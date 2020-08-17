---
title: Alta disponibilidad de máquinas virtuales de Azure para SAP NetWeaver en SLES | Microsoft Docs
description: Guía de alta disponibilidad para SAP NetWeaver en SUSE Linux Enterprise Server para SAP Applications
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 5e514964-c907-4324-b659-16dd825f6f87
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/04/2020
ms.author: radeltch
ms.openlocfilehash: 01a450c045c996cdcb49b8fbfdf1ce572ee2d1df
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87760607"
---
# <a name="high-availability-for-sap-netweaver-on-azure-vms-on-suse-linux-enterprise-server-for-sap-applications"></a>Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[2205917]:https://launchpad.support.sap.com/#/notes/2205917
[1944799]:https://launchpad.support.sap.com/#/notes/1944799
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[1410736]:https://launchpad.support.sap.com/#/notes/1410736

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[suse-ha-guide]:https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
[suse-drbd-guide]:https://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha_techguides/book_sleha_techguides.html
[suse-ha-12sp3-relnotes]:https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP3/

[template-multisid-xscs]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[template-converged]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-converged-md%2Fazuredeploy.json
[template-file-server]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-file-server-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability.md
[nfs-ha]:high-availability-guide-suse-nfs.md

En este artículo se describe cómo implementar y configurar las máquinas virtuales, y cómo instalar el marco del clúster y un sistema SAP NetWeaver 7.50 de alta disponibilidad.
En las configuraciones, comandos de instalación, etc. de ejemplo, Se usa el número de instancia 00 de ASCS, el número de instancia 02 de ERS y NW1 del identificador de sistema de SAP. En los nombres de los recursos (por ejemplo, máquinas virtuales o redes virtuales) del ejemplo se dan por sentado que ha usado la [plantilla convergente][template-converged] con NW1 del identificador de sistema de SAP para crear los recursos.

Lea primero las notas y los documentos de SAP siguientes:

* Nota de SAP [1928533][1928533], que incluye:
  * La lista de tamaños de máquina virtual de Azure que se admiten para la implementación del software de SAP
  * Información importante sobre la capacidad para los tamaños de máquina virtual de Azure
  * Software de SAP admitido y combinaciones de sistema operativo y base de datos
  * Versión del kernel de SAP requerida para Windows y Linux en Microsoft Azure

* La nota de SAP [2015553][2015553] enumera los requisitos previos para las implementaciones de software de SAP admitidas por SAP en Azure.
* La nota de SAP [2205917][2205917] contiene configuraciones recomendadas del sistema operativo para SUSE Linux Enterprise Server para SAP Applications
* La nota de SAP [1944799][1944799] contiene guías de SAP HANA para SUSE Linux Enterprise Server para SAP Applications
* La nota de SAP [2178632][2178632] contiene información detallada sobre todas las métricas de supervisión notificadas para SAP en Azure.
* La nota de SAP [2191498][2191498] incluye la versión de SAP Host Agent necesaria para Linux en Azure.
* La nota de SAP [2243692][2243692] incluye información acerca de las licencias de SAP en Linux en Azure.
* La nota de SAP [1984787][1984787] incluye información general sobre SUSE Linux Enterprise Server 12.
* La nota de SAP [1999351][1999351] contiene más soluciones de problemas de la extensión de supervisión mejorada de Azure para SAP.
* La [WIKI de la comunidad SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contiene todas las notas de SAP que se necesitan para Linux.
* [Planeación e implementación de Azure Virtual Machines para SAP en Linux][planning-guide]
* [Implementación de Azure Virtual Machines para SAP en Linux][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP en Linux][dbms-guide]
* [Guías de procedimientos recomendados de SUSE SAP HA][suse-ha-guide] Las guías contienen toda la información necesaria para la configuración de Netweaver HA y la replicación del sistema de SAP HANA en el entorno local. Use estas guías como base de referencia general. Proporcionan información mucho más detallada.
* [Notas de la versión de la Extensión 12 SP3 de alta disponibilidad para SUSE][suse-ha-12sp3-relnotes]

## <a name="overview"></a>Información general

Para lograr alta disponibilidad, SAP NetWeaver requiere un servidor NFS. El servidor NFS está configurado en un clúster distinto y lo pueden usar varios sistemas SAP.

![Información general sobre la alta disponibilidad de SAP NetWeaver](./media/high-availability-guide-suse/ha-suse.png)

El servidor NFS, SAP NetWeaver ASCS, SAP NetWeaver SCS, SAP NetWeaver ERS y la base de datos SAP HANA usan direcciones IP virtuales y nombre de host virtual. En Azure, se requiere un equilibrador de carga para usar una dirección IP virtual. Se recomienda usar [Standard Load Balancer](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md). En la lista siguiente se muestra la configuración del equilibrador de carga (A)SCS y ERS.

### <a name="ascs"></a>(A)SCS

* Configuración de front-end
  * Dirección IP 10.0.0.7
* Puerto de sondeo
  * Puerto 620<strong>&lt;nr&gt;</strong>
* Reglas de equilibrio de carga
  * Si usa Standard Load Balancer, seleccione **Puertos HA**
  * Si usa Basic Load Balancer, cree reglas de equilibrio de carga para los puertos siguientes
    * 32<strong>&lt;nr&gt;</strong> TCP
    * 36<strong>&lt;nr&gt;</strong> TCP
    * 39<strong>&lt;nr&gt;</strong> TCP
    * 81<strong>&lt;nr&gt;</strong> TCP
    * 5<strong>&lt;nr&gt;</strong>13 TCP
    * 5<strong>&lt;nr&gt;</strong>14 TCP
    * 5<strong>&lt;nr&gt;</strong>16 TCP

### <a name="ers"></a>ERS

* Configuración de front-end
  * Dirección IP 10.0.0.8
* Puerto de sondeo
  * Puerto 621<strong>&lt;nr&gt;</strong>
* Reglas de equilibrio de carga.
  * Si usa Standard Load Balancer, seleccione **Puertos HA**
  * Si usa Basic Load Balancer, cree reglas de equilibrio de carga para los puertos siguientes
    * 32<strong>&lt;nr&gt;</strong> TCP
    * 33<strong>&lt;nr&gt;</strong> TCP
    * 5<strong>&lt;nr&gt;</strong>13 TCP
    * 5<strong>&lt;nr&gt;</strong>14 TCP
    * 5<strong>&lt;nr&gt;</strong>16 TCP

* Configuración de back-end
  * Se conecta a interfaces de red principales de todas las máquinas que deben ser parte del clúster (A)SCS/ERS


## <a name="setting-up-a-highly-available-nfs-server"></a>Configuración de un servidor NFS de alta disponibilidad

SAP NetWeaver requiere un almacenamiento compartido para el directorio de transporte y perfil. Lea [Alta disponibilidad para NFS en máquinas virtuales de Azure en SUSE Linux Enterprise Server][nfs-ha] para obtener información acerca de cómo configurar un servidor NFS para SAP NetWeaver.

## <a name="setting-up-ascs"></a>Configuración de (A)SCS

Puede usar una plantilla de Azure de GitHub para implementar todos los recursos de Azure necesarios, como máquinas virtuales, conjunto de disponibilidad y equilibrador de carga, o puede implementar los recursos manualmente.

### <a name="deploy-linux-via-azure-template"></a>Implementación de Linux mediante una plantilla de Azure

Azure Marketplace contiene una imagen de SUSE Linux Enterprise Server para SAP Applications 12 que puede usar para implementar nuevas máquinas virtuales. La imagen de Marketplace contiene el agente de recursos para SAP NetWeaver.

Para implementar todos los recursos necesarios, puede usar una de las plantillas de inicio rápido de GitHub. La plantilla implementa las máquinas virtuales, el equilibrador de carga, el conjunto de disponibilidad, etc. Siga estos pasos para implementar la plantilla:

1. Abra la [plantilla ASCS/SCS de varios SID][template-multisid-xscs] o la [plantilla convergente][template-converged] en Azure Portal. 
   La plantilla ASCS/SCS solo crea las reglas de equilibrio de carga para las instancias ASCS/SCS y ERS (solo Linux) de SAP NetWeaver, mientras que la plantilla convergente también crea las reglas de equilibrio de carga para una base de datos (por ejemplo, Microsoft SQL Server o SAP HANA). Si tiene previsto instalar un sistema basado en SAP NetWeaver y desea instalar la base de datos en las mismas máquinas, use la [plantilla convergente][template-converged].
1. Escriba los siguientes parámetros:
   1. Prefijo de recurso (solo la plantilla de varios ASCS/SCS de varios SID)  
      Escriba el prefijo que desea usar. El valor se usa como prefijo de los recursos que se implementan.
   3. Identificador del sistema SAP (solo la plantilla convergente)  
      Escriba el identificador del sistema SAP que se va a instalar. El identificador se usa como prefijo de los recursos que se implementan.
   4. Tipo de pila  
      Seleccione el tipo de la pila de SAP NetWeaver
   5. Tipo de sistema operativo  
      Seleccione una de las distribuciones de Linux. En este ejemplo, seleccione SLES 12 BYOS
   6. Tipo de base de datos  
      Seleccione HANA
   7. Tamaño del sistema SAP.  
      La cantidad de SAPS que proporciona el sistema nuevo. Si no está seguro de cuántos SAP necesita el sistema, consulte con el integrador de sistemas o el asociado tecnológico de SAP
   8. Disponibilidad del sistema  
      Seleccione alta disponibilidad
   9. Nombre de usuario y contraseña del administrador  
      Se crea un usuario nuevo que se puede usar para iniciar sesión en la máquina.
   10. Identificador de subred  
   Si quiere implementar la máquina virtual en una red virtual existente en la que tiene una subred definida a la que se debe asignar la máquina virtual, asigne un nombre al identificador de esa subred específica. El identificador suele tener este aspecto: /subscriptions/ **&lt;id. de suscripción&gt;** /resourceGroups/ **&lt;nombre del grupo de recursos&gt;** /providers/Microsoft.Network/virtualNetworks/ **&lt;nombre de red virtual&gt;** /subnets/ **&lt;nombre de subred&gt;**

### <a name="deploy-linux-manually-via-azure-portal"></a>Implementación manual de Linux mediante Azure Portal

En primer lugar, debe crear las máquinas virtuales de este clúster NFS. Después, creará un equilibrador de carga y usará las máquinas virtuales del grupo de servidores back-end.

1. Creación de un grupo de recursos
1. Creación de una red virtual
1. Creación de un conjunto de disponibilidad  
   Establecimiento del dominio máximo de actualización
1. Creación de la máquina virtual 1  
   Use al menos SLES4SAP 12 SP1; en este ejemplo se usa la imagen SLES4SAP 12 SP1 https://portal.azure.com/#create/SUSE.SUSELinuxEnterpriseServerforSAPApplications12SP1PremiumImage-ARM.  
   Se usa SLES para SAP Applications 12 SP1  
   Seleccione el conjunto de disponibilidad creado anteriormente.  
1. Creación de la máquina virtual 2  
   Use al menos SLES4SAP 12 SP1; en este ejemplo se usa la imagen SLES4SAP 12 SP1 https://portal.azure.com/#create/SUSE.SUSELinuxEnterpriseServerforSAPApplications12SP1PremiumImage-ARM.  
   Se usa SLES para SAP Applications 12 SP1  
   Seleccione el conjunto de disponibilidad creado anteriormente.  
1. Agregue al menos un disco de datos a ambas máquinas virtuales.  
   Los discos de datos se usan para el directorio /usr/sap/`<SAPSID`>.
1. Cree un equilibrador de carga (interno, estándar):  
   1. Creación de las direcciones IP de front-end
      1. Dirección IP 10.0.0.7 de ASCS
         1. Abra el equilibrador de carga, seleccione el grupo de direcciones IP de front-end y haga clic en Agregar
         1. Escriba el nombre del nuevo grupo de direcciones IP de front-end (por ejemplo, **nw1-ascs-frontend**).
         1. Configure la asignación como estática y escriba la dirección IP (por ejemplo **10.0.0.7**).
         1. Haga clic en Aceptar
      1. Dirección IP 10.0.0.8 para ASCS ERS
         * Repita los pasos anteriores para crear una dirección IP para el ERS (por ejemplo, **10.0.0.8** y **nw1-aers-backend**).
   1. Creación del grupo de servidores back-end
      1. Abra el equilibrador de carga, seleccione los grupos de back-end y haga clic en Agregar
      1. Escriba el nombre del nuevo grupo de servidores back-end (por ejemplo, **nw1-backend**).
      1. Haga clic en Agregar una máquina virtual.
      1. Seleccione Máquina virtual.
      1. Seleccione las máquinas virtuales del clúster de (A)SCS y sus direcciones IP.
      1. Haga clic en Agregar
   1. Creación de los sondeos de estado
      1. Puerto 620**00** para ASCS
         1. Abra el equilibrador de carga, seleccione los sondeos de estado y haga clic en Agregar
         1. Escriba el nombre del nuevo sondeo de estado (por ejemplo, **nw1-ascs-hp**).
         1. Seleccione TCP como protocolo, puerto 620**00**, y mantenga el intervalo de 5 y el umbral incorrecto 2.
         1. Haga clic en Aceptar
      1. Puerto 621**02** para ASCS ERS
         * Repita los pasos anteriores para crear un sondeo de estado para ERS (por ejemplo 621**02** y **nw1-aers-hp**).
   1. Reglas de equilibrio de carga.
      1. Reglas de equilibrio de carga para ASCS
         1. Abra el equilibrador de carga, seleccione las reglas de equilibrio de carga y haga clic en Agregar.
         1. Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, **nw1-lb-ascs**)
         1. Seleccione la dirección IP de front-end, el grupo de servidores back-end y el sondeo de estado creados anteriormente (por ejemplo, **nw1-ascs-frontend**, **nw1-backend** y **nw1-ascs-hp**).
         1. Seleccione **Puertos HA**
         1. Aumente el tiempo de espera de inactividad a 30 minutos
         1. **Asegúrese de habilitar la dirección IP flotante**
         1. Haga clic en Aceptar
         * Repita los pasos anteriores para crear reglas de equilibrio de carga para ERS (por ejemplo **nw1-lb-ers**)
1. Como alternativa, si el escenario requiere un equilibrador de carga básico (interno), siga estos pasos:  
   1. Creación de las direcciones IP de front-end
      1. Dirección IP 10.0.0.7 de ASCS
         1. Abra el equilibrador de carga, seleccione el grupo de direcciones IP de front-end y haga clic en Agregar
         1. Escriba el nombre del nuevo grupo de direcciones IP de front-end (por ejemplo, **nw1-ascs-frontend**).
         1. Configure la asignación como estática y escriba la dirección IP (por ejemplo **10.0.0.7**).
         1. Haga clic en Aceptar
      1. Dirección IP 10.0.0.8 para ASCS ERS
         * Repita los pasos anteriores para crear una dirección IP para el ERS (por ejemplo, **10.0.0.8** y **nw1-aers-frontend**).
   1. Creación del grupo de servidores back-end
      1. Abra el equilibrador de carga, seleccione los grupos de back-end y haga clic en Agregar
      1. Escriba el nombre del nuevo grupo de servidores back-end (por ejemplo, **nw1-backend**).
      1. Haga clic en Agregar una máquina virtual.
      1. Seleccione el conjunto de disponibilidad que creó anteriormente
      1. Seleccione las máquinas virtuales del clúster (A)SCS.
      1. Haga clic en Aceptar
   1. Creación de los sondeos de estado
      1. Puerto 620**00** para ASCS
         1. Abra el equilibrador de carga, seleccione los sondeos de estado y haga clic en Agregar
         1. Escriba el nombre del nuevo sondeo de estado (por ejemplo, **nw1-ascs-hp**).
         1. Seleccione TCP como protocolo, puerto 620**00**, y mantenga el intervalo de 5 y el umbral incorrecto 2.
         1. Haga clic en Aceptar
      1. Puerto 621**02** para ASCS ERS
         * Repita los pasos anteriores para crear un sondeo de estado para ERS (por ejemplo 621**02** y **nw1-aers-hp**).
   1. Reglas de equilibrio de carga.
      1. TCP 32**00** para ASCS
         1. Abra el equilibrador de carga, seleccione las reglas de equilibrio de carga y haga clic en Agregar.
         1. Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, **nw1-lb-3200**).
         1. Seleccione la dirección IP de front-end, el grupo de servidores back-end y el sondeo de estado que creó anteriormente (por ejemplo, **nw1-ascs-frontend**).
         1. Conserve el protocolo **TCP** y escriba el puerto **3200**.
         1. Aumente el tiempo de espera de inactividad a 30 minutos
         1. **Asegúrese de habilitar la dirección IP flotante**
         1. Haga clic en Aceptar
      1. Puertos adicionales para ASCS
         * Repita los pasos anteriores para los puertos 36**00**, 39**00**, 81**00**, 5**00**13, 5**00**14, 5**00**16 y TCP para ASCS.
      1. Puertos adicionales para ASCS ERS
         * Repita los pasos anteriores para los puertos 33**02**, 5**02**13, 5**02**14, 5**02**16 y TCP para ASCS ERS

> [!Note]
> Cuando las máquinas virtuales sin direcciones IP públicas se colocan en el grupo de back-end de Standard Load Balancer interno (sin dirección IP pública), no hay conectividad saliente de Internet, a menos que se realice una configuración adicional para permitir el enrutamiento a puntos de conexión públicos. Para obtener más información sobre cómo obtener conectividad saliente, vea [Conectividad de punto de conexión público para máquinas virtuales con Azure Standard Load Balancer en escenarios de alta disponibilidad de SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

> [!IMPORTANT]
> No habilite las marcas de tiempo TCP en VM de Azure que se encuentren detrás de Azure Load Balancer. Si habilita las marcas de tiempo TCP provocará un error en los sondeos de estado. Establezca el parámetro **net.ipv4.tcp_timestamps** a **0**. Consulte [Sondeos de estado de Load Balancer](../../../load-balancer/load-balancer-custom-probe-overview.md) para obtener más información.

### <a name="create-pacemaker-cluster"></a>Creación del clúster de Pacemaker

Siga los pasos que se describen en [Configuración de Pacemaker en SUSE Linux Enterprise Server en Azure](high-availability-guide-suse-pacemaker.md) para crear un clúster de Pacemaker básico para este servidor (A)SCS.

### <a name="installation"></a>Instalación

Los elementos siguientes tienen el prefijo **[A]** : aplicable a todos los nodos, **[1]** : aplicable solo al nodo 1 o **[2]** : aplicable solo al nodo 2.

1. **[A]** Instale el conector de SUSE

   <pre><code>sudo zypper install sap-suse-cluster-connector
   </code></pre>

   > [!NOTE]
   > El problema conocido con el uso de un guion en los nombres de host se ha corregido con la versión **3.1.1** del paquete **sap-suse-cluster-connector**. Si utiliza nodos de clúster con un guion en el nombre de host, asegúrese de usar al menos la versión 3.1.1 del paquete sap-suse-cluster-connector. Si lo hace, el clúster no funcionará. 

   Asegúrese de que instaló la nueva versión del conector de clúster SUSE SAP. La antigua se llamaba sap_suse_cluster_connector y la nueva se llama **sap-suse-cluster-connector**.

   ```
   sudo zypper info sap-suse-cluster-connector
   
   Information for package sap-suse-cluster-connector:
   ---------------------------------------------------
   Repository     : SLE-12-SP3-SAP-Updates
   Name           : sap-suse-cluster-connector
   <b>Version        : 3.0.0-2.2</b>
   Arch           : noarch
   Vendor         : SUSE LLC <https://www.suse.com/>
   Support Level  : Level 3
   Installed Size : 41.6 KiB
   <b>Installed      : Yes</b>
   Status         : up-to-date
   Source package : sap-suse-cluster-connector-3.0.0-2.2.src
   Summary        : SUSE High Availability Setup for SAP Products
   ```

1. **[A]** Actualice los agentes de recursos de SAP  
   
   Se requiere una revisión del paquete de agentes de recursos para usar la configuración nueva, que es la que describe este artículo. Puede usar el comando siguiente para comprobar si la revisión ya está instalada

   <pre><code>sudo grep 'parameter name="IS_ERS"' /usr/lib/ocf/resource.d/heartbeat/SAPInstance
   </code></pre>

   La salida debe ser similar a

   <pre><code>&lt;parameter name="IS_ERS" unique="0" required="0"&gt;
   </code></pre>

   Si el comando grep no encuentra el parámetro IS_ERS, necesita instalar la revisión que aparece en [la página de descarga de SUSE](https://download.suse.com/patch/finder/#bu=suse&familyId=&productId=&dateRange=&startDate=&endDate=&priority=&architecture=&keywords=resource-agents)

   <pre><code># example for patch for SLES 12 SP1
   sudo zypper in -t patch SUSE-SLE-HA-12-SP1-2017-885=1
   # example for patch for SLES 12 SP2
   sudo zypper in -t patch SUSE-SLE-HA-12-SP2-2017-886=1
   </code></pre>

1. **[A]** Configure la resolución nombres de host

   Puede usar un servidor DNS o modificar /etc/hosts en todos los nodos. En este ejemplo se muestra cómo utilizar el archivo /etc/hosts.
   Reemplace la dirección IP y el nombre de host en los siguientes comandos

   <pre><code>sudo vi /etc/hosts
   </code></pre>

   Inserte las siguientes líneas en /etc/hosts. Cambie la dirección IP y el nombre de host para que coincida con su entorno   

   <pre><code># IP address of the load balancer frontend configuration for NFS
   <b>10.0.0.4 nw1-nfs</b>
   # IP address of the load balancer frontend configuration for SAP NetWeaver ASCS
   <b>10.0.0.7 nw1-ascs</b>
   # IP address of the load balancer frontend configuration for SAP NetWeaver ASCS ERS
   <b>10.0.0.8 nw1-aers</b>
   # IP address of the load balancer frontend configuration for database
   <b>10.0.0.13 nw1-db</b>
   </code></pre>

## <a name="prepare-for-sap-netweaver-installation"></a>Preparación de la instalación de SAP NetWeaver

1. **[A]** Cree los directorios compartidos

   <pre><code>sudo mkdir -p /sapmnt/<b>NW1</b>
   sudo mkdir -p /usr/sap/trans
   sudo mkdir -p /usr/sap/<b>NW1</b>/SYS
   sudo mkdir -p /usr/sap/<b>NW1</b>/ASCS<b>00</b>
   sudo mkdir -p /usr/sap/<b>NW1</b>/ERS<b>02</b>
   
   sudo chattr +i /sapmnt/<b>NW1</b>
   sudo chattr +i /usr/sap/trans
   sudo chattr +i /usr/sap/<b>NW1</b>/SYS
   sudo chattr +i /usr/sap/<b>NW1</b>/ASCS<b>00</b>
   sudo chattr +i /usr/sap/<b>NW1</b>/ERS<b>02</b>
   </code></pre>

1. **[A]** Configure autofs

   <pre><code>sudo vi /etc/auto.master
   
   # Add the following line to the file, save and exit
   +auto.master
   /- /etc/auto.direct
   </code></pre>

   Cree un archivo con

   <pre><code>sudo vi /etc/auto.direct
   
   # Add the following lines to the file, save and exit
   /sapmnt/<b>NW1</b> -nfsvers=4,nosymlink,sync <b>nw1-nfs</b>:/<b>NW1</b>/sapmntsid
   /usr/sap/trans -nfsvers=4,nosymlink,sync <b>nw1-nfs</b>:/<b>NW1</b>/trans
   /usr/sap/<b>NW1</b>/SYS -nfsvers=4,nosymlink,sync <b>nw1-nfs</b>:/<b>NW1</b>/sidsys
   </code></pre>

   Reinicie autofs para montar los recursos compartidos nuevos

   <pre><code>sudo systemctl enable autofs
   sudo service autofs restart
   </code></pre>

1. **[A]** Configure el archivo de intercambio

   <pre><code>sudo vi /etc/waagent.conf
   
   # Set the property ResourceDisk.EnableSwap to y
   # Create and use swapfile on resource disk.
   ResourceDisk.EnableSwap=<b>y</b>
   
   # Set the size of the SWAP file with property ResourceDisk.SwapSizeMB
   # The free space of resource disk varies by virtual machine size. Make sure that you do not set a value that is too big. You can check the SWAP space with command swapon
   # Size of the swapfile.
   ResourceDisk.SwapSizeMB=<b>2000</b>
   </code></pre>

   Reinicie el agente para activar el cambio

   <pre><code>sudo service waagent restart
   </code></pre>


### <a name="installing-sap-netweaver-ascsers"></a>Instalación de SAP NetWeaver ASCS/ERS

1. **[1]** Cree un recurso IP virtual y un sondeo de estado para la instancia de ASCS

   > [!IMPORTANT]
   > Pruebas recientes han mostrado situaciones en las que netcat deja de responder a las solicitudes debido al trabajo pendiente y a su limitación para controlar solo una conexión. El recurso netcat deja de escuchar las solicitudes del equilibrador de carga de Azure y la dirección IP flotante deja de estar disponible.  
   > En el caso de los clústeres de Pacemaker existentes, en el pasado se recomendaba reemplazar netcat por socat. Actualmente se recomienda usar el agente de recursos azure-lb, que forma parte de los agentes de recursos de paquetes, con los siguientes requisitos de versión de paquete:
   > - En el caso de SLES 12 SP4/SP5, la versión debe ser, al menos, resource-agents-4.3.018.a7fb5035-3.30.1.  
   > - Para SLES 15/15 SP1, la versión debe ser al menos resource-agents-4.3.0184.6ee15eb2-4.13.1.  
   >
   > Tenga en cuenta que el cambio requerirá un breve tiempo de inactividad.  
   > En el caso de los clústeres de Pacemaker existentes, si la configuración ya se ha cambiado para usar socat, como se describe en [Protección de la detección del equilibrador de carga de Azure](https://www.suse.com/support/kb/doc/?id=7024128), no hay ningún requisito para cambiar inmediatamente al agente de recursos de azure-lb.

   <pre><code>sudo crm node standby <b>nw1-cl-1</b>
   
   sudo crm configure primitive fs_<b>NW1</b>_ASCS Filesystem device='<b>nw1-nfs</b>:/<b>NW1</b>/ASCS' directory='/usr/sap/<b>NW1</b>/ASCS<b>00</b>' fstype='nfs4' \
     op start timeout=60s interval=0 \
     op stop timeout=60s interval=0 \
     op monitor interval=20s timeout=40s
   
   sudo crm configure primitive vip_<b>NW1</b>_ASCS IPaddr2 \
     params ip=<b>10.0.0.7</b> cidr_netmask=<b>24</b> \
     op monitor interval=10 timeout=20
   
   sudo crm configure primitive nc_<b>NW1</b>_ASCS azure-lb port=620<b>00</b>
   
   sudo crm configure group g-<b>NW1</b>_ASCS fs_<b>NW1</b>_ASCS nc_<b>NW1</b>_ASCS vip_<b>NW1</b>_ASCS \
      meta resource-stickiness=3000
   </code></pre>

   Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.

   <pre><code>sudo crm_mon -r
   
   # Node nw1-cl-1: standby
   # <b>Online: [ nw1-cl-0 ]</b>
   # 
   # Full list of resources:
   # 
   # stonith-sbd     (stonith:external/sbd): <b>Started nw1-cl-0</b>
   #  Resource Group: g-NW1_ASCS
   #      fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    <b>Started nw1-cl-0</b>
   #      nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      <b>Started nw1-cl-0</b>
   #      vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       <b>Started nw1-cl-0</b>
   </code></pre>

1. **[1]** Instale SAP NetWeaver ASCS  

   Instale SAP NetWeaver ASCS como raíz en el primer nodo mediante un nombre de host virtual que se asigna a la dirección IP de la configuración de front-end del equilibrador de carga para ASCS; por ejemplo, <b>nw1-ascs</b>, <b>10.0.0.7</b>. Especifique también el número de instancia que usó para el sondeo del equilibrador de carga; por ejemplo, <b>00</b>.

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b>
   </code></pre>

   Si se produce un error en la instalación para crear una subcarpeta en/usr/sap/**NW1**/ASCS**00**, pruebe a establecer el propietario y el grupo de la carpeta ASCS**00** e inténtelo de nuevo.

   <pre><code>chown nw1adm /usr/sap/<b>NW1</b>/ASCS<b>00</b>
   chgrp sapsys /usr/sap/<b>NW1</b>/ASCS<b>00</b>
   </code></pre>

1. **[1]** Cree un recurso IP virtual y un sondeo de estado para la instancia de ERS

   <pre><code>sudo crm node online <b>nw1-cl-1</b>
   sudo crm node standby <b>nw1-cl-0</b>
   
   sudo crm configure primitive fs_<b>NW1</b>_ERS Filesystem device='<b>nw1-nfs</b>:/<b>NW1</b>/ASCSERS' directory='/usr/sap/<b>NW1</b>/ERS<b>02</b>' fstype='nfs4' \
     op start timeout=60s interval=0 \
     op stop timeout=60s interval=0 \
     op monitor interval=20s timeout=40s
   
   sudo crm configure primitive vip_<b>NW1</b>_ERS IPaddr2 \
     params ip=<b>10.0.0.8</b> cidr_netmask=<b>24</b> \
     op monitor interval=10 timeout=20
   
   sudo crm configure primitive nc_<b>NW1</b>_ERS azure-lb port=621<b>02</b>
   
   sudo crm configure group g-<b>NW1</b>_ERS fs_<b>NW1</b>_ERS nc_<b>NW1</b>_ERS vip_<b>NW1</b>_ERS
   </code></pre>

   Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.

   <pre><code>sudo crm_mon -r
   
   # Node <b>nw1-cl-0: standby</b>
   # <b>Online: [ nw1-cl-1 ]</b>
   # 
   # Full list of resources:
   #
   # stonith-sbd     (stonith:external/sbd): <b>Started nw1-cl-1</b>
   #  Resource Group: g-NW1_ASCS
   #      fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    <b>Started nw1-cl-1</b>
   #      nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      <b>Started nw1-cl-1</b>
   #      vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       <b>Started nw1-cl-1</b>
   #  Resource Group: g-NW1_ERS
   #      fs_NW1_ERS (ocf::heartbeat:Filesystem):    <b>Started nw1-cl-1</b>
   #      nc_NW1_ERS (ocf::heartbeat:azure-lb):      <b>Started nw1-cl-1</b>
   #      vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       <b>Started nw1-cl-1</b>
   </code></pre>

1. **[2]** Instale SAP NetWeaver ERS

   Instale SAP NetWeaver ERS como raíz en el segundo nodo mediante un nombre de host virtual que se asigna a la dirección IP de la configuración de front-end del equilibrador de carga para ERS (por ejemplo, <b>nw1-aers</b>, <b>10.0.0.8</b>) y el número de instancia que usó para el sondeo del equilibrador de carga (por ejemplo, <b>02</b>).

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b>
   </code></pre>

   > [!NOTE]
   > Use SWPM SP 20 PL 05 o superior. Las versiones inferiores no establecen correctamente los permisos y se producirá un error de instalación.

   Si se produce un error en la instalación para crear una subcarpeta en/usr/sap/**NW1**/ERS**00**, pruebe a establecer el propietario y el grupo de la carpeta ERS**02** e inténtelo de nuevo.

   <pre><code>chown nw1adm /usr/sap/<b>NW1</b>/ERS<b>02</b>
   chgrp sapsys /usr/sap/<b>NW1</b>/ERS<b>02</b>
   </code></pre>


1. **[1]** Adapte los perfiles de instancias ASCS/SCS y ERS
 
   * Perfil ASCS/SCS

   <pre><code>sudo vi /sapmnt/<b>NW1</b>/profile/<b>NW1</b>_<b>ASCS00</b>_<b>nw1-ascs</b>
   
   # Change the restart command to a start command
   #Restart_Program_01 = local $(_EN) pf=$(_PF)
   Start_Program_01 = local $(_EN) pf=$(_PF)
   
   # Add the following lines
   service/halib = $(DIR_CT_RUN)/saphascriptco.so
   service/halib_cluster_connector = /usr/bin/sap_suse_cluster_connector
   
   # Add the keep alive parameter, if using ENSA1
   enque/encni/set_so_keepalive = true
   </code></pre>

   En el caso de ENSA1 y ENSA2, asegúrese de que los parámetros del sistema operativo `keepalive` se establecen tal y como se describe en la nota de SAP [1410736](https://launchpad.support.sap.com/#/notes/1410736).    

   * Perfil ERS

   <pre><code>sudo vi /sapmnt/<b>NW1</b>/profile/<b>NW1</b>_ERS<b>02</b>_<b>nw1-aers</b>
   
   # Change the restart command to a start command
   #Restart_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   Start_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   
   # Add the following lines
   service/halib = $(DIR_CT_RUN)/saphascriptco.so
   service/halib_cluster_connector = /usr/bin/sap_suse_cluster_connector
   
   # remove Autostart from ERS profile
   # Autostart = 1
   </code></pre>

1. **[A]** Configure la conexión persistente

   La comunicación entre el servidor de aplicaciones de SAP NetWeaver y ASCS/SCS se enruta a través de un equilibrador de carga de software. El equilibrador de carga desconecta las conexiones inactivas después de un tiempo de expiración que se puede configurar. Para evitar esto, debe establecer un parámetro en el perfil de SAP NetWeaver ASCS/SCS y cambiar la configuración del sistema Linux. Para más información, lea la [nota de SAP 1410736][1410736].

   <pre><code># Change the Linux system configuration
   sudo sysctl net.ipv4.tcp_keepalive_time=120
   </code></pre>

1. **[A]** Configure los usuarios de SAP después de la instalación

   <pre><code># Add sidadm to the haclient group
   sudo usermod -aG haclient <b>nw1</b>adm
   </code></pre>

1. **[1]** Agregue los servicios SAP de ASCS y ERS al archivo sapservice

   Agregue la entrada del servicio ASCS al segundo nodo y copie la entrada del servicio ERS al primer nodo.

   <pre><code>cat /usr/sap/sapservices | grep ASCS<b>00</b> | sudo ssh <b>nw1-cl-1</b> "cat >>/usr/sap/sapservices"
   sudo ssh <b>nw1-cl-1</b> "cat /usr/sap/sapservices" | grep ERS<b>02</b> | sudo tee -a /usr/sap/sapservices
   </code></pre>

1. **[1]** Cree los recursos de clúster de SAP

Si usa la arquitectura de servidor 1 de puesta en cola (ENSA1), defina los recursos tal como se indica:

   <pre><code>sudo crm configure property maintenance-mode="true"
   
   sudo crm configure primitive rsc_sap_<b>NW1</b>_ASCS<b>00</b> SAPInstance \
    operations \$id=rsc_sap_<b>NW1</b>_ASCS<b>00</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>NW1</b>_ASCS<b>00</b>_<b>nw1-ascs</b> START_PROFILE="/sapmnt/<b>NW1</b>/profile/<b>NW1</b>_ASCS<b>00</b>_<b>nw1-ascs</b>" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 failure-timeout=60 migration-threshold=1 priority=10
   
   sudo crm configure primitive rsc_sap_<b>NW1</b>_ERS<b>02</b> SAPInstance \
    operations \$id=rsc_sap_<b>NW1</b>_ERS<b>02</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>NW1</b>_ERS<b>02</b>_<b>nw1-aers</b> START_PROFILE="/sapmnt/<b>NW1</b>/profile/<b>NW1</b>_ERS<b>02</b>_<b>nw1-aers</b>" AUTOMATIC_RECOVER=false IS_ERS=true \
    meta priority=1000
   
   sudo crm configure modgroup g-<b>NW1</b>_ASCS add rsc_sap_<b>NW1</b>_ASCS<b>00</b>
   sudo crm configure modgroup g-<b>NW1</b>_ERS add rsc_sap_<b>NW1</b>_ERS<b>02</b>
   
   sudo crm configure colocation col_sap_<b>NW1</b>_no_both -5000: g-<b>NW1</b>_ERS g-<b>NW1</b>_ASCS
   sudo crm configure location loc_sap_<b>NW1</b>_failover_to_ers rsc_sap_<b>NW1</b>_ASCS<b>00</b> rule 2000: runs_ers_<b>NW1</b> eq 1
   sudo crm configure order ord_sap_<b>NW1</b>_first_start_ascs Optional: rsc_sap_<b>NW1</b>_ASCS<b>00</b>:start rsc_sap_<b>NW1</b>_ERS<b>02</b>:stop symmetrical=false
   
   sudo crm node online <b>nw1-cl-0</b>
   sudo crm configure property maintenance-mode="false"
   </code></pre>

  SAP introdujo una opción de compatibilidad con el servidor 2 de puesta en cola, incluida la replicación, a partir de la versión de SAP NW 7.52. A partir de la plataforma ABAP (versión 1809), el servidor 2 de puesta en cola está instalado de forma predeterminada. Consulte la nota de SAP [2630416](https://launchpad.support.sap.com/#/notes/2630416) para consultar la compatibilidad con el servidor 2 de puesta en cola.
Si usa la arquitectura de servidor 2 de puesta en cola ([ENSA2](https://help.sap.com/viewer/cff8531bc1d9416d91bb6781e628d4e0/1709%20001/en-US/6d655c383abf4c129b0e5c8683e7ecd8.html)), defina los recursos tal como se indica:

<pre><code>sudo crm configure property maintenance-mode="true"
   
   sudo crm configure primitive rsc_sap_<b>NW1</b>_ASCS<b>00</b> SAPInstance \
    operations \$id=rsc_sap_<b>NW1</b>_ASCS<b>00</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>NW1</b>_ASCS<b>00</b>_<b>nw1-ascs</b> START_PROFILE="/sapmnt/<b>NW1</b>/profile/<b>NW1</b>_ASCS<b>00</b>_<b>nw1-ascs</b>" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000
   
   sudo crm configure primitive rsc_sap_<b>NW1</b>_ERS<b>02</b> SAPInstance \
    operations \$id=rsc_sap_<b>NW1</b>_ERS<b>02</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>NW1</b>_ERS<b>02</b>_<b>nw1-aers</b> START_PROFILE="/sapmnt/<b>NW1</b>/profile/<b>NW1</b>_ERS<b>02</b>_<b>nw1-aers</b>" AUTOMATIC_RECOVER=false IS_ERS=true 
   
   sudo crm configure modgroup g-<b>NW1</b>_ASCS add rsc_sap_<b>NW1</b>_ASCS<b>00</b>
   sudo crm configure modgroup g-<b>NW1</b>_ERS add rsc_sap_<b>NW1</b>_ERS<b>02</b>
   
   sudo crm configure colocation col_sap_<b>NW1</b>_no_both -5000: g-<b>NW1</b>_ERS g-<b>NW1</b>_ASCS
   sudo crm configure order ord_sap_<b>NW1</b>_first_start_ascs Optional: rsc_sap_<b>NW1</b>_ASCS<b>00</b>:start rsc_sap_<b>NW1</b>_ERS<b>02</b>:stop symmetrical=false
   
   sudo crm node online <b>nw1-cl-0</b>
   sudo crm configure property maintenance-mode="false"
   </code></pre>

  Si está actualizando desde una versión anterior y va a cambiar al servidor 2 de puesta en cola, consulte la nota de SAP [2641019](https://launchpad.support.sap.com/#/notes/2641019). 

   Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.


   <pre><code>sudo crm_mon -r
   
   # Online: <b>[ nw1-cl-0 nw1-cl-1 ]</b>
   #
   # Full list of resources:
   #
   # stonith-sbd     (stonith:external/sbd): <b>Started nw1-cl-1</b>
   #  Resource Group: g-NW1_ASCS
   #      fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    <b>Started nw1-cl-1</b>
   #      nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      <b>Started nw1-cl-1</b>
   #      vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       <b>Started nw1-cl-1</b>
   #      rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   <b>Started nw1-cl-1</b>
   #  Resource Group: g-NW1_ERS
   #      fs_NW1_ERS (ocf::heartbeat:Filesystem):    <b>Started nw1-cl-0</b>
   #      nc_NW1_ERS (ocf::heartbeat:azure-lb):      <b>Started nw1-cl-0</b>
   #      vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       <b>Started nw1-cl-0</b>
   #      rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   <b>Started nw1-cl-0</b>
   </code></pre>

## <a name="sap-netweaver-application-server-preparation"></a><a name="2d6008b0-685d-426c-b59e-6cd281fd45d7"></a>Preparación del servidor de aplicaciones de SAP NetWeaver

Algunas bases de datos requieren que la instalación de la instancia de base de datos se ejecute en un servidor de aplicaciones. Prepare las máquinas virtuales del servidor de aplicaciones para poder usarlas en estos casos.

En los pasos siguientes se supone que instala el servidor de aplicaciones en un servidor distinto de los servidores ASCS/SCS y HANA. De lo contrario, no se necesitan algunos de los pasos que aparecen a continuación (como configurar la resolución de nombres de host).

1. Configuración del sistema operativo

   Reduzca el tamaño de la caché de datos incorrectos. Para más información, consulte [Low write performance on SLES 11/12 servers with large RAM](https://www.suse.com/support/kb/doc/?id=7010287) (Bajo rendimiento de escritura en servidores SLES 11/12 con RAM grande).

   <pre><code>sudo vi /etc/sysctl.conf

   # Change/set the following settings
   vm.dirty_bytes = 629145600
   vm.dirty_background_bytes = 314572800
   </code></pre>

1. Configure la resolución de nombres de host

   Puede usar un servidor DNS o modificar /etc/hosts en todos los nodos. En este ejemplo se muestra cómo utilizar el archivo /etc/hosts.
   Reemplace la dirección IP y el nombre de host en los siguientes comandos

   ```bash
   sudo vi /etc/hosts
   ```

   Inserte las siguientes líneas en /etc/hosts. Cambie la dirección IP y el nombre de host para que coincida con su entorno

   <pre><code># IP address of the load balancer frontend configuration for NFS
   <b>10.0.0.4 nw1-nfs</b>
   # IP address of the load balancer frontend configuration for SAP NetWeaver ASCS/SCS
   <b>10.0.0.7 nw1-ascs</b>
   # IP address of the load balancer frontend configuration for SAP NetWeaver ERS
   <b>10.0.0.8 nw1-aers</b>
   # IP address of the load balancer frontend configuration for database
   <b>10.0.0.13 nw1-db</b>
   # IP address of all application servers
   <b>10.0.0.20 nw1-di-0</b>
   <b>10.0.0.21 nw1-di-1</b>
   </code></pre>

1. Cree el directorio sapmnt

   <pre><code>sudo mkdir -p /sapmnt/<b>NW1</b>
   sudo mkdir -p /usr/sap/trans

   sudo chattr +i /sapmnt/<b>NW1</b>
   sudo chattr +i /usr/sap/trans
   </code></pre>

1. Configure autofs

   <pre><code>sudo vi /etc/auto.master
   
   # Add the following line to the file, save and exit
   +auto.master
   /- /etc/auto.direct
   </code></pre>

   Cree un nuevo archivo con

   <pre><code>sudo vi /etc/auto.direct
   
   # Add the following lines to the file, save and exit
   /sapmnt/<b>NW1</b> -nfsvers=4,nosymlink,sync <b>nw1-nfs</b>:/<b>NW1</b>/sapmntsid
   /usr/sap/trans -nfsvers=4,nosymlink,sync <b>nw1-nfs</b>:/<b>NW1</b>/trans
   </code></pre>

   Reinicie autofs para montar los recursos compartidos nuevos

   <pre><code>sudo systemctl enable autofs
   sudo service autofs restart
   </code></pre>

1. Configure el archivo de intercambio

   <pre><code>sudo vi /etc/waagent.conf
   
   # Set the property ResourceDisk.EnableSwap to y
   # Create and use swapfile on resource disk.
   ResourceDisk.EnableSwap=<b>y</b>
   
   # Set the size of the SWAP file with property ResourceDisk.SwapSizeMB
   # The free space of resource disk varies by virtual machine size. Make sure that you do not set a value that is too big. You can check the SWAP space with command swapon
   # Size of the swapfile.
   ResourceDisk.SwapSizeMB=<b>2000</b>
   </code></pre>

   Reinicie el agente para activar el cambio

   <pre><code>sudo service waagent restart
   </code></pre>

## <a name="install-database"></a>Instalar la base de datos

En este ejemplo, SAP NetWeaver se instala en SAP HANA. En esta instalación puede usar todas las bases de datos admitidas. Para más información acerca de cómo instalar SAP HANA en Azure, consulte [Alta disponibilidad de SAP HANA en Azure Virtual Machines (VM)][sap-hana-ha]. Para ver una lista de las bases de datos admitidas, consulte la [nota de SAP 1928533][1928533].

1. Ejecute la instalación de la instancia de base de datos de SAP.

   Instale la instancia de base de datos de SAP NetWeaver como raíz con un nombre de host virtual que se asigna a la dirección IP de la configuración de front-end del equilibrador de carga para la base de datos, por ejemplo, <b>nw1-db</b> y <b>10.0.0.13</b>.

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b>
   </code></pre>

## <a name="sap-netweaver-application-server-installation"></a>Instalación del servidor de aplicaciones de SAP NetWeaver

Siga estos pasos para instalar un servidor de aplicaciones de SAP.

1. Preparación del servidor de aplicaciones

   Siga los pasos descritos en el capítulo [Preparación del servidor de aplicaciones de SAP NetWeaver](high-availability-guide-suse.md#2d6008b0-685d-426c-b59e-6cd281fd45d7) anterior para preparar el servidor de aplicaciones.

1. Instale el servidor de aplicaciones de SAP NetWeaver

   Instale un servidor de aplicaciones de SAP NetWeaver principal o adicional.

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b>
   </code></pre>

1. Actualice el almacenamiento seguro de SAP HANA

   Actualice el almacenamiento seguro de SAP HANA que apunte al nombre virtual de la configuración de la replicación del sistema SAP HANA.

   Ejecute el siguiente comando para mostrar las entradas:
   <pre><code>hdbuserstore List
   </code></pre>

   Se deberían mostrar todas las entradas y deberían ser parecidas a estas:
   <pre><code>DATA FILE       : /home/nw1adm/.hdb/nw1-di-0/SSFS_HDB.DAT
   KEY FILE        : /home/nw1adm/.hdb/nw1-di-0/SSFS_HDB.KEY
   
   KEY DEFAULT
     ENV : 10.0.0.14:<b>30313</b>
     USER: <b>SAPABAP1</b>
     DATABASE: <b>HN1</b>
   </code></pre>

   El resultado muestra que la dirección IP de la entrada predeterminada apunta a la máquina virtual y no a la dirección IP del equilibrador de carga. Esta entrada debe modificarse para que apunte al nombre de host virtual del equilibrador de carga. Asegúrese de usar el mismo puerto (**30313** en la salida anterior) y el mismo nombre de base de datos (**HN1** en la salida anterior).

   <pre><code>su - <b>nw1</b>adm
   hdbuserstore SET DEFAULT <b>nw1-db:30313@HN1</b> <b>SAPABAP1</b> <b>&lt;password of ABAP schema&gt;</b>
   </code></pre>

## <a name="test-the-cluster-setup"></a>Prueba de la configuración del clúster

Las siguientes pruebas son una copia de los casos de prueba de las guías de procedimientos recomendados de SUSE. Para su comodidad, se han copiado en este artículo. Además, lea siempre las guías de procedimientos recomendados y realice todas las pruebas adicionales que puedan haberse agregado.

1. Prueba de HAGetFailoverConfig, HACheckConfig y HACheckFailoverConfig

   Ejecute los siguientes comandos como \<sapsid>adm en el nodo donde se ejecuta actualmente la instancia de ASCS. Si los comandos producen un error de memoria insuficiente, los guiones en el nombre de host pueden ser la causa. Se trata de un problema conocido que SUSE corregirá en el paquete sap-suse-cluster-connector.

   <pre><code>nw1-cl-0:nw1adm 54> sapcontrol -nr <b>00</b> -function HAGetFailoverConfig
   
   # 15.08.2018 13:50:36
   # HAGetFailoverConfig
   # OK
   # HAActive: TRUE
   # HAProductVersion: Toolchain Module
   # HASAPInterfaceVersion: Toolchain Module (sap_suse_cluster_connector 3.0.1)
   # HADocumentation: https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
   # HAActiveNode:
   # HANodes: nw1-cl-0, nw1-cl-1
   
   nw1-cl-0:nw1adm 55> sapcontrol -nr 00 -function HACheckConfig
   
   # 15.08.2018 14:00:04
   # HACheckConfig
   # OK
   # state, category, description, comment
   # SUCCESS, SAP CONFIGURATION, Redundant ABAP instance configuration, 2 ABAP instances detected
   # SUCCESS, SAP CONFIGURATION, Redundant Java instance configuration, 0 Java instances detected
   # SUCCESS, SAP CONFIGURATION, Enqueue separation, All Enqueue server separated from application server
   # SUCCESS, SAP CONFIGURATION, MessageServer separation, All MessageServer separated from application server
   # SUCCESS, SAP CONFIGURATION, ABAP instances on multiple hosts, ABAP instances on multiple hosts detected
   # SUCCESS, SAP CONFIGURATION, Redundant ABAP SPOOL service configuration, 2 ABAP instances with SPOOL service detected
   # SUCCESS, SAP STATE, Redundant ABAP SPOOL service state, 2 ABAP instances with active SPOOL service detected
   # SUCCESS, SAP STATE, ABAP instances with ABAP SPOOL service on multiple hosts, ABAP instances with active ABAP SPOOL service on multiple hosts detected
   # SUCCESS, SAP CONFIGURATION, Redundant ABAP BATCH service configuration, 2 ABAP instances with BATCH service detected
   # SUCCESS, SAP STATE, Redundant ABAP BATCH service state, 2 ABAP instances with active BATCH service detected
   # SUCCESS, SAP STATE, ABAP instances with ABAP BATCH service on multiple hosts, ABAP instances with active ABAP BATCH service on multiple hosts detected
   # SUCCESS, SAP CONFIGURATION, Redundant ABAP DIALOG service configuration, 2 ABAP instances with DIALOG service detected
   # SUCCESS, SAP STATE, Redundant ABAP DIALOG service state, 2 ABAP instances with active DIALOG service detected
   # SUCCESS, SAP STATE, ABAP instances with ABAP DIALOG service on multiple hosts, ABAP instances with active ABAP DIALOG service on multiple hosts detected
   # SUCCESS, SAP CONFIGURATION, Redundant ABAP UPDATE service configuration, 2 ABAP instances with UPDATE service detected
   # SUCCESS, SAP STATE, Redundant ABAP UPDATE service state, 2 ABAP instances with active UPDATE service detected
   # SUCCESS, SAP STATE, ABAP instances with ABAP UPDATE service on multiple hosts, ABAP instances with active ABAP UPDATE service on multiple hosts detected
   # SUCCESS, SAP STATE, SCS instance running, SCS instance status ok
   # SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version (nw1-ascs_NW1_00), SAPInstance includes is-ers patch
   # SUCCESS, SAP CONFIGURATION, Enqueue replication (nw1-ascs_NW1_00), Enqueue replication enabled
   # SUCCESS, SAP STATE, Enqueue replication state (nw1-ascs_NW1_00), Enqueue replication active
   
   nw1-cl-0:nw1adm 56> sapcontrol -nr 00 -function HACheckFailoverConfig
   
   # 15.08.2018 14:04:08
   # HACheckFailoverConfig
   # OK
   # state, category, description, comment
   # SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version, SAPInstance includes is-ers patch
   </code></pre>

1. Migración manual de la instancia de ASCS

   Estado del recurso antes de iniciar la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-0
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
   </code></pre>

   Ejecute los siguientes comandos como raíz para migrar la instancia de ASCS.

   <pre><code>nw1-cl-0:~ # crm resource migrate rsc_sap_NW1_ASCS00 force
   # INFO: Move constraint created for rsc_sap_NW1_ASCS00
   
   nw1-cl-0:~ # crm resource unmigrate rsc_sap_NW1_ASCS00
   # INFO: Removed migration constraints for rsc_sap_NW1_ASCS00
   
   # Remove failed actions for the ERS that occurred as part of the migration
   nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ERS02
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-0
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

1. Prueba de HAFailoverToNode

   Estado del recurso antes de iniciar la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-0
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

   Ejecute los siguientes comandos como \<sapsid>administrador para migrar la instancia de ASCS.

   <pre><code>nw1-cl-0:nw1adm 55> sapcontrol -nr 00 -host nw1-ascs -user nw1adm &lt;password&gt; -function HAFailoverToNode ""
   
   # run as root
   # Remove failed actions for the ERS that occurred as part of the migration
   nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ERS02
   # Remove migration constraints
   nw1-cl-0:~ # crm resource clear rsc_sap_NW1_ASCS00
   #INFO: Removed migration constraints for rsc_sap_NW1_ASCS00
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-0
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
   </code></pre>

1. Simulación de bloqueo de nodo

   Estado del recurso antes de iniciar la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-0
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
   </code></pre>

   Ejecute el siguiente comando como raíz en el nodo donde se ejecuta la instancia de ASCS

   <pre><code>nw1-cl-0:~ # echo b > /proc/sysrq-trigger
   </code></pre>

   Si usa SBD, Pacemaker no se iniciará automáticamente en el nodo terminado. El estado después de que el nodo se inicia de nuevo debe parecerse a este.

   <pre><code>Online: [ nw1-cl-1 ]
   OFFLINE: [ nw1-cl-0 ]
   
   Full list of resources:
   
   stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
   
   Failed Actions:
   * rsc_sap_NW1_ERS02_monitor_11000 on nw1-cl-1 'not running' (7): call=219, status=complete, exitreason='none',
       last-rc-change='Wed Aug 15 14:38:38 2018', queued=0ms, exec=0ms
   </code></pre>

   Use los siguientes comandos para iniciar Pacemaker en el nodo terminado, limpiar los mensajes de SBD y limpiar los recursos con error.

   <pre><code># run as root
   # list the SBD device(s)
   nw1-cl-0:~ # cat /etc/sysconfig/sbd | grep SBD_DEVICE=
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116;/dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1;/dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3"
   
   nw1-cl-0:~ # sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116 -d /dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1 -d /dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3 message nw1-cl-0 clear
   
   nw1-cl-0:~ # systemctl start pacemaker
   nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ASCS00
   nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ERS02
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

1. Prueba del reinicio manual de la instancia de ASCS

   Estado del recurso antes de iniciar la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

   Cree un bloqueo de puesta en cola; por ejemplo, edite un usuario en la transacción su01. Ejecute los comandos siguientes como \<sapsid>adm en el nodo donde se ejecuta la instancia de ASCS. Los comandos detendrán la instancia de ASCS y la volverán a iniciar. Si usa la arquitectura deL servidor 1 de puesta en cola, es posible que el bloqueo de puesta en cola se pierda en esta prueba. Si usa la arquitectura del servidor 2 de puesta en cola, se conservará la puesta en cola. 

   <pre><code>nw1-cl-1:nw1adm 54> sapcontrol -nr 00 -function StopWait 600 2
   </code></pre>

   Ahora, la instancia de ASCS se deshabilitará en Pacemaker.

   <pre><code>rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Stopped (disabled)
   </code></pre>

   Vuelva a iniciar la instancia de ASCS en el mismo nodo.

   <pre><code>nw1-cl-1:nw1adm 54> sapcontrol -nr 00 -function StartWait 600 2
   </code></pre>

   El bloqueo de puesta en cola de la transacción su01 se perderá y el back-end se habrá restablecido. Estado del recurso después de la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

1. Terminación del proceso del servidor de mensajes

   Estado del recurso antes de iniciar la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

   Ejecute los siguientes comandos como raíz para identificar el proceso del servidor de mensajes y terminarlo.

   <pre><code>nw1-cl-1:~ # pgrep ms.sapNW1 | xargs kill -9
   </code></pre>

   Si solo termina el servidor de mensajes una vez, se reiniciará mediante sapstart. Si lo termina con bastante frecuencia, Pacemaker acabará moviendo la instancia de ASCS al otro nodo. Ejecute los siguientes comandos como raíz para limpiar el estado del recurso de la instancia de ASCS y ERS después de la prueba.

   <pre><code>nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ASCS00
   nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ERS02
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
   </code></pre>

1. Terminación del proceso del servidor de puesta en cola

   Estado del recurso antes de iniciar la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
   </code></pre>

   Ejecute los siguientes comandos como raíz en el nodo donde se ejecuta la instancia de ASCS para terminar el servidor de puesta en cola.

   <pre><code>nw1-cl-0:~ # pgrep en.sapNW1 | xargs kill -9
   </code></pre>

   La instancia de ASCS conmutará inmediatamente por error al otro nodo. La instancia de ERS también conmutará por error después de iniciarse la instancia de ASCS. Ejecute los siguientes comandos como raíz para limpiar el estado del recurso de la instancia de ASCS y ERS después de la prueba.

   <pre><code>nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ASCS00
   nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ERS02
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

1. Terminación del proceso del servidor de replicación

   Estado del recurso antes de iniciar la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

   Ejecute el siguiente comando como raíz en el nodo donde se ejecuta la instancia de ERS para terminar el proceso del servidor de replicación de puesta en cola.

   <pre><code>nw1-cl-0:~ # pgrep er.sapNW1 | xargs kill -9
   </code></pre>

   Si solo ejecuta el comando una vez, sapstart reiniciará el proceso. Si lo ejecuta con bastante frecuencia, sapstart no reiniciará el proceso y el recurso estará en estado detenido. Ejecute los siguientes comandos como raíz para limpiar el estado del recurso de la instancia de ERS después de la prueba.

   <pre><code>nw1-cl-0:~ # crm resource cleanup rsc_sap_NW1_ERS02
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

1. Terminación del proceso sapstartsrv de puesta en cola

   Estado del recurso antes de iniciar la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

   Ejecute los siguientes comandos como raíz en el nodo donde se ejecuta ASCS.

   <pre><code>nw1-cl-1:~ # pgrep -fl ASCS00.*sapstartsrv
   # 59545 sapstartsrv
   
   nw1-cl-1:~ # kill -9 59545
   </code></pre>

   El proceso sapstartsrv siempre se debe reiniciar con el agente de recursos de Pacemaker. Estado del recurso después de la prueba:

   <pre><code>stonith-sbd     (stonith:external/sbd): Started nw1-cl-1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started nw1-cl-1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started nw1-cl-1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started nw1-cl-1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started nw1-cl-1
    Resource Group: g-NW1_ERS
        fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started nw1-cl-0
        nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started nw1-cl-0
        vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started nw1-cl-0
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started nw1-cl-0
   </code></pre>

## <a name="next-steps"></a>Pasos siguientes

* [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications: guía de varios SID](./high-availability-guide-suse-multi-sid.md)
* [Planeamiento e implementación de Azure Virtual Machines para SAP][planning-guide]
* [Implementación de Azure Virtual Machines para SAP][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP][dbms-guide]
* Para más información sobre cómo establecer la alta disponibilidad y planear la recuperación ante desastres de SAP HANA en Azure Virtual Machines, consulte [Alta disponibilidad de SAP HANA en las máquinas virtuales de Azure][sap-hana-ha].
