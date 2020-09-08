---
title: 'Preguntas frecuentes: Azure VMware Solution by CloudSimple'
description: Estas preguntas más frecuentes sobre Azure VMware Solution by CloudSimple incluyen preguntas sobre conectividad, redes, seguridad, almacenamiento, integración de Azure y otras.
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/15/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 0db8c13a7467ed864c0845319b37f958f60fa4d9
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88140894"
---
# <a name="frequently-asked-questions-about-vmware-solution-by-cloudsimple"></a>Preguntas frecuentes sobre VMware Solution by CloudSimple

## <a name="cloudsimple-service"></a>Servicio CloudSimple

**¿Qué es Azure VMware Solution by CloudSimple?**

Azure VMware Solution by CloudSimple transforma y extiende las cargas de trabajo de VMware a nubes privadas y dedicadas en Azure en cuestión de minutos. CloudSimple se encarga del aprovisionamiento, la administración de la infraestructura y la orquestación de las cargas de trabajo entre el entorno local y Azure. Como las aplicaciones se ejecutan de la misma forma en el entorno local y en Azure, se beneficia de la elasticidad y los servicios de la nube sin la complejidad de rediseñar las aplicaciones. CloudSimple reduce el costo total de propiedad con un modelo de consumo en la nube que proporciona aprovisionamiento a petición, pagos en función del crecimiento y optimización de la capacidad.  Para escenarios, ventajas y características, consulte [¿Qué es la solución de VMware en Azure de CloudSimple?](cloudsimple-vmware-solutions-overview.md)

**¿Qué es una nube privada de CloudSimple?**

Una nube privada de CloudSimple es una nube privada y dedicada formada por un proceso de alto rendimiento, almacenamiento y un entorno de red implementado en la infraestructura de Microsoft Azure (hardware y espacio de centro de datos) en ubicaciones de Azure.  Una nube privada proporciona una plataforma como servicio nativa de VMware. En términos de VMware, cada nube privada contiene exactamente un servidor de vCenter Server. vCenter Server administra varios nodos de ESXi incluidos en uno o varios clústeres de vSphere, junto con el almacenamiento virtual SAN (vSAN) correspondiente. Un servicio de CloudSimple puede contener varias nubes privadas en la suscripción de Azure.  Para más información, consulte [Introducción a la nube privada de CloudSimple](cloudsimple-private-cloud.md).

**¿Dónde está disponible el servicio de CloudSimple?**

CloudSimple está disponible en las regiones Este de EE. UU., Oeste de EE. UU. y Oeste de Europa (y próximamente en otras regiones).

**¿Cómo habilito mi suscripción para CloudSimple?**

Póngase en contacto con su representante de cuenta de Microsoft en [azurevmwaresales@microsoft.com](mailto:azurevmwaresales@microsoft.com) para habilitar la suscripción para el servicio de CloudSimple. En el correo electrónico, especifique el identificador de suscripción para el que quiera habilitar el servicio CloudSimple.  

**¿Cómo se accede al portal de CloudSimple?**

Puede acceder al portal de CloudSimple desde Azure Portal.  Para más información, consulte [Access the VMware Solution by CloudSimple portal from the Azure portal](access-cloudsimple-portal.md) (Acceso a la solución de VMware mediante el portal de CloudSimple desde Azure Portal).

**¿Cómo se puede aumentar la capacidad de una nube privada?**

Para aumentar la capacidad, adquiera nodos adicionales desde Azure Portal y úselos para expandir la nube privada desde el portal de CloudSimple.  Puede agregar nodos adicionales a un clúster de vSphere existente o a uno nuevo.  Para más información, consulte [Expand a CloudSimple Private Cloud](expand-private-cloud.md) (Expansión de una nube privada de CloudSimple).

**¿Qué ocurre con mi nube privada durante el mantenimiento?**

CloudSimple proporciona una notificación varios días antes del mantenimiento programado.  El mantenimiento sin disrupciones para garantizar la disponibilidad de la nube privada.  El mantenimiento puede ser de uno de los tipos siguientes:

* **Infraestructura de CloudSimple**.  La infraestructura de CloudSimple está diseñada para tener alta disponibilidad.  Durante este tipo de mantenimiento, los componentes redundantes se actualizan de uno en uno para evitar cualquier interrupción del servicio. Tendrá acceso continuado a la instancia de vCenter de la nube privada, a todas las máquinas virtuales, conexión a Internet desde la nube privada y conexiones al entorno local o a Azure.
* **Portal de CloudSimple**. Durante este tipo de mantenimiento, algunas características del portal de CloudSimple pueden estar deshabilitadas o inaccesibles.  La notificación anterior al mantenimiento incluye detalles sobre las limitaciones de las características mientras este se realiza.

## <a name="connectivity"></a>Conectividad

**¿Cuáles son mis opciones de conectividad en la red de la región de CloudSimple?**

CloudSimple proporciona las siguientes opciones de conectividad para conectarse a la red de la región de CloudSimple. Se pueden usar varias opciones al mismo tiempo.

* **Conexión de ExpressRoute desde el centro de datos local a la red de la región de CloudSimple**. Es una conexión privada segura de alta velocidad y latencia baja que usa Global Reach para conectar el circuito ExpressRoute local con el circuito ExpressRoute de CloudSimple. Para instrucciones para establecer la conexión, consulte [Connect from on-premises to CloudSimple using ExpressRoute](on-premises-connection.md) (Conexión desde el entorno local a CloudSimple mediante ExpressRoute).
* **Conexión de ExpressRoute desde la red virtual de Azure a la red de la región de CloudSimple**. Es una conexión privada segura de alta velocidad y latencia baja que usa puertas de enlace de red virtual para conectar la red virtual de Azure con el circuito ExpressRoute de CloudSimple. Para instrucciones para establecer la conexión, consulte [Connect your CloudSimple Private Cloud environment to the Azure virtual network using ExpressRoute](azure-expressroute-connection.md) (Conexión del entorno de nube privada de CloudSimple a la red virtual de Azure mediante ExpressRoute).
* **Conexión VPN de sitio a sitio desde el centro de datos local a la red de la región de CloudSimple**. Es una red virtual privada segura desde el dispositivo VPN local hasta la región de la nube privada de CloudSimple.  Para más información, consulte [Set up VPN gateways on CloudSimple network](vpn-gateway.md) (Configuración de puertas de enlace de VPN en la red de CloudSimple).

**¿Cómo me conecto a una nube privada?**

Puede ver los detalles de la nube privada en el portal de CloudSimple. Para conectarse a la instancia de vCenter correspondiente a la nube privada, perimero asegúrese de que se establece una conexión de red mediante VPN de sitio a sitio, VPN de punto a sitio o ExpressRoute. A continuación, inicie el portal de CloudSimple desde Azure Portal y haga clic en **Launch vSphere Client** (Iniciar cliente vSphere) en la página principal o en la página de detalles de la nube privada.

**¿Qué ventaja tienen los circuitos ExpressRoute?**

Un circuito Azure ExpressRoute es una conexión segura de alta velocidad y baja latencia.  CloudSimple proporciona un circuito ExpressRoute dedicado por cada región y cliente.  Con este circuito, puede establecer una conexión segura desde el entorno local o la suscripción de Azure.

**¿Cuáles son los costos de red para conectarse a CloudSimple?  ¿Se aplican cargos de salida entre CloudSimple y Azure o entre regiones?**

No hay ningún cargo de CloudSimple por la salida de red.  Se aplican las tarifas estándar de Azure a todo el tráfico de salida desde la red virtual o desde el circuito ExpressRoute local.

## <a name="networking"></a>Redes

**¿Qué características de red están disponibles para la nube privada?**

Puede aprovisionar redes VLAN (y sus subredes) y tablas de firewall, y asignar direcciones IP públicas que a su vez se asignen a una máquina virtual que se ejecute en la nube privada. Para más información sobre las características de redes, consulte [Introducción a las redes VLAN y las subredes](cloudsimple-vlans-subnets.md), [Firewall tables overview](cloudsimple-firewall-tables.md) (Introducción a las tablas de firewall) e [Introducción a la dirección IP pública](cloudsimple-public-ip-address.md).

**¿Cómo configuro otras subredes para las aplicaciones en la nube privada?**

Puede crear redes VLAN en la nube privada desde el portal de CloudSimple.  Después de crear la VLAN, puede crear un grupo de puertos distribuido en la instancia de vCenter de la nube privada mediante la red VLAN y crear máquinas virtuales conectadas al grupo de puertos distribuido.  Puede habilitar tablas de firewall para la subred o la red VLAN y definir reglas de firewall para proteger el tráfico.

**¿Qué configuración de firewall está disponible para las nubes privadas?**

Puede configurar reglas para el tráfico norte-sur y este-oeste.  Las reglas se definen en una tabla de firewall.  La tabla de firewall se puede conectar a redes VLAN de la nube privada.  Para más información, consulte [Set up firewal tables and rules for Private Clouds](firewall.md) (Configuración de tablas de firewall y reglas para las nubes privadas).

**¿Puedo asignar direcciones IP públicas a las máquinas virtuales en el entorno de nube privada?**

En el portal de CloudSimple, puede asignar una nueva dirección IP pública y asociarla a una dirección IP privada de una máquina virtual o un dispositivo.  En el portal también puede crear reglas de firewall o aplicar reglas de firewall existentes para permitir el tráfico desde puertos y direcciones IP específicos. Para más información, consulte [Allocate public IP addresses for Private Cloud environment](public-ips.md) (Asignación de direcciones IP públicas para un entorno de nube privada).

## <a name="security"></a>Seguridad

**¿Cuáles son mis opciones de seguridad en CloudSimple?**

CloudSimple proporciona las siguientes características de seguridad para proteger el entorno de nube privada:

* **Cifrado de datos en reposo**. Puede cifrar los datos en reposo que residen en el almacenamiento vSAN en la nube privada. vSAN admite los servidores de administración de claves externas, que se puede implementar en el entorno local o la red virtual de Azure.  Para más información, consulte [Configure vSAN encryption for your CloudSimple Private Cloud](vsan-encryption.md) (Configuración de cifrado de vSAN para la nube privada de CloudSimple).
* **Seguridad de red**. Controle el flujo de tráfico con reglas de firewall que se apliquen entre su nube privada e Internet, la nube privada y el entorno local, o dentro de subredes de la nube privada.
* **Conexión privada y segura**. Se establece una conexión privada y segura entre la red local y la suscripción de Azure.

## <a name="compute"></a>Proceso

**¿Qué tipos de hosts están disponibles?**

CloudSimple ofrece estos tipos de host:

* **Nodo CS28:** CPU: 2x 2,2 GHz, 28 núcleos en total, 48 HT.  RAM: 256 GB.  Almacenamiento: Caché NVMe de 1600 GB, 5760 GB de datos (All-Flash). Red: NIC de 4x25 Gbe
* **Nodo CS36:** CPU 2x 2,3 GHz, 36 núcleos en total, 72 HT.  RAM: 512 GB.  Almacenamiento: Caché NVMe de 3200 GB, 11520 GB de datos (All-Flash).  Red: NIC de 4x25 Gbe
* **Nodo CS36m:** CPU 2x 2,3 GHz, 36 núcleos en total, 72 HT.  RAM: 576 GB.  Almacenamiento: Caché NVMe de 3200 GB, 13360 GB de datos (All-Flash).  Red: NIC de 4x25 Gbe

**¿Cómo se controlan los errores de hardware?**

Toda la infraestructura de CloudSimple se supervisa de forma continua mediante la plataforma CloudSimple y los equipos de operaciones de servicio.  Si se detecta un error de hardware, se agrega un nodo nuevo a la nube privada y se elimina el nodo con el error.

## <a name="storage"></a>Storage

**¿Qué tipo de almacenamiento se admite en una nube privada?**

CloudSimple ofrece almacenamiento vSAN de VMware All-Flash con cada nube privada.  Cada instancia de vSphere se crea con su propio almacén de datos vSAN.  Para más información, consulte [Private Cloud VMware components - vSAN storage](vmware-components.md#vsan-storage) (Componentes de VMware para la nube privada: almacenamiento de vSAN).

**¿Se admite el cifrado de datos?**
Sí.  Puede configurar el almacenamiento de vSAN en la nube privada para usar un servidor de administración de claves (KMS) que se implemente en el entorno local o en Azure para cifrar los datos almacenados en vSAN.

**¿Cómo se controlan los discos con errores?**

CloudSimple supervisa de forma continua todos los componentes de hardware de la nube privada.  Si se detecta un error de disco o se identifica un disco con errores a partir de la heurística, se agrega automáticamente un nodo nuevo a la nube privada.  El nodo con un disco con errores se quita de la nube privada.

## <a name="vmware"></a>VMware

**¿Cómo se realiza la carga y la migración a gran escala de aplicaciones y datos desde el entorno local?**

CloudSimple proporciona una solución VMware vSphere nativa.  Todas las herramientas de VMware para la migración de datos de forma masiva se pueden utilizar con la nube privada.  Las opciones incluyen:

* VMware HCX para la migración masiva de datos.
* Migración en frío de datos mediante Storage vMotion desde el entorno local a CloudSimple.

**¿Puedo instalar cualquier herramienta de VMware?**

CloudSimple proporciona una solución VMware vSphere nativa.  Todas las herramientas de VMware que se usan para administrar el entorno de vSphere local se pueden usar en CloudSimple.  CloudSimple admite un modelo traiga su propia licencia (BYOL) para la instalación de herramientas de VMware.

**¿Cómo se administran las actualizaciones?**

CloudSimple administra y actualiza todos los componentes de infraestructura de la nube privada de forma transparente y sin interrupciones.  Las actualizaciones y revisiones de seguridad publicadas por VMware o los proveedores de infraestructura se programan para la actualización en cuanto cumplan las condiciones de CloudSimple.

CloudSimple no realiza actualizaciones de las aplicaciones instaladas en la nube privada.

## <a name="azure-integration"></a>Integración de Azure

**¿Qué servicios de Azure se admiten?**

CloudSimple ofrece conexión de Azure ExpressRoute a la suscripción en Azure.  Todos los servicios que se ejecutan en la suscripción pueden conectarse a la nube privada.  Algunos ejemplos son:

* **Azure Active Directory** como origen de identidades para la instancia de vCenter de CloudSimple.
* **Azure Storage** para almacenar copias de seguridad, imágenes y otros datos de la nube privada.
* **Aplicaciones híbridas** con una arquitectura de aplicación que abarque las nubes públicas y privadas.  Por ejemplo, puede crear servidores web en Azure que accedan a servidores de aplicaciones y base de datos en la nube privada.
* **Azure Monitor** y **Azure Security Center** para las cargas de trabajo que se ejecutan en VMware admiten el registro, las métricas de rendimiento y la administración de seguridad.

**¿Cómo se asignan los inquilinos de VMware a Azure?**

CloudSimple proporciona la capacidad única de administrar las máquinas virtuales de VMware en la nube privada desde Azure Portal.  El administrador global puede asignar a la suscripción un grupo de recursos de vCenter configurado con las restricciones de recursos que quiera.  

**¿Qué ventajas de licencia obtengo con Azure?**

Con CloudSimple puede aprovechar la ventaja del uso híbrido de Azure y ahorrar hasta un 90 % en las licencias. Esta ventaja conserva su inversión en las licencias de Microsoft y reduce el costo total de propiedad de otras soluciones en la nube. También obtiene actualizaciones de seguridad ampliadas para Windows Server 2008 y Microsoft SQL Server 2008.  El modelo BYOL (traiga su propia licencia) ayuda a mantener los costos reducidos para las aplicaciones comunes como Veeam y Zerto.  
