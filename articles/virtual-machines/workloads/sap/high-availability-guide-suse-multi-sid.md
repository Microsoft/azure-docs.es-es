---
title: 'Alta disponibilidad de las máquinas virtuales de Azure para SAP NetWeaver en SLES: guía de varios SID | Microsoft Docs'
description: Guía de varios SID de alta disponibilidad para SAP NetWeaver en SUSE Linux Enterprise Server para SAP Applications
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
ms.openlocfilehash: 16c37c1492b042e9f2f19e631f7801bfbed2d247
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87761219"
---
# <a name="high-availability-for-sap-netweaver-on-azure-vms-on-suse-linux-enterprise-server-for-sap-applications-multi-sid-guide"></a>Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications: guía de varios SID

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-sap-applications-azure]:https://www.netapp.com/us/media/tr-4746.pdf

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

En este artículo se describe cómo implementar varios sistemas de alta disponibilidad de SAP NetWeaver o S4HANA (es decir, varios SID) en un clúster de dos nodos en máquinas virtuales de Azure con SUSE Linux Enterprise Server para SAP Applications.  

En las configuraciones de ejemplo, los comandos de instalación, etc., se implementan tres sistemas SAP NetWeaver 7.50 en un único clúster de alta disponibilidad de dos nodos. Los SID de los sistemas SAP son los siguientes:
* **NW1**: Número de instancia de ASCS **00** y nombre de host virtual **msnw1ascs**; número de instancia de ERS **02** y nombre de host virtual **msnw1ers**.  
* **NW2**: Número de instancia de ASCS **10** y nombre de host virtual **msnw2ascs**; número de instancia de ERS **12** y nombre de host virtual **msnw2ers**.  
* **NW3**: Número de instancia de ASCS **20** y nombre de host virtual **msnw3ascs**; número de instancia de ERS **22** y nombre de host virtual **msnw3ers**.  

El artículo no trata la capa de base de datos ni la implementación de los recursos compartidos NFS de SAP. En los ejemplos de este artículo, usamos nombres virtuales nw2-nfs para los recursos compartidos NFS de NW2 y nw3-nfs para los recursos compartidos NFS de NW3, suponiendo que se haya implementado el clúster NFS.  

Antes de comenzar, consulte las siguientes notas y documentos de SAP:

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
* [Guía de clústeres de varios SID de SUSE para SLES 12 y SLES 15](https://documentation.suse.com/sbp/all/html/SBP-SAP-MULTI-SID/index.html)
* [Aplicaciones de NetApp SAP en Microsoft Azure que usan Azure NetApp Files][anf-sap-applications-azure]
## <a name="overview"></a>Información general

Se debe ajustar el tamaño de las máquinas virtuales que participan en el clúster para poder ejecutar todos los recursos en caso de que se produzca la conmutación por error. Cada SID de SAP puede conmutar por error de forma independiente en el clúster de alta disponibilidad de varios SID.  Si usa vallado de SBD, los dispositivos SBD pueden compartirse entre varios clústeres.  

Para lograr alta disponibilidad, SAP NetWeaver requiere recursos compartidos NFS disponibles. En este ejemplo se supone que los recursos compartidos NFS de SAP se hospedan en un [servidor de archivos NFS](./high-availability-guide-suse-nfs.md) de alta disponibilidad, que puede usarse en varios sistemas SAP. O bien, los recursos compartidos se implementan en [volúmenes de NFS de Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).  

![Información general sobre la alta disponibilidad de SAP NetWeaver](./media/high-availability-guide-suse/ha-suse-multi-sid.png)

> [!IMPORTANT]
> La compatibilidad con la agrupación en clústeres de varios SID de SAP ASCS/ERS con SUSE Linux como sistema operativo invitado en las máquinas virtuales de Azure se limita a **cinco** SIDs de SAP en un mismo clúster. Cada nuevo SID aumenta la complejidad. No **se admite** una combinación de SAP Enqueue Replication Server 1 y SAP Enqueue Replication Server 2 en el mismo clúster. La agrupación en clústeres de varios SID describe la instalación de varias instancias de SAP ASCS/ERS con SID diferentes en un clúster de Pacemaker. Actualmente, la agrupación en clústeres de varios SID solo se admite para ASCS/ERS.  

> [!TIP]
> La agrupación en clústeres de varios SID de SAP ASCS/ERS es una solución de gran complejidad. Su implementación resulta más compleja. También implica un mayor esfuerzo administrativo al ejecutar actividades de mantenimiento (como la aplicación de revisiones del SO). Antes de comenzar la implementación real, dedique tiempo a planear cuidadosamente la implementación y todos los componentes implicados como máquinas virtuales, montajes NFS, VIP, configuraciones de equilibrador de carga, etc.  

El servidor NFS, SAP NetWeaver ASCS, SAP NetWeaver SCS, SAP NetWeaver ERS y la base de datos SAP HANA usan direcciones IP virtuales y nombre de host virtual. En Azure, se requiere un equilibrador de carga para usar una dirección IP virtual. Se recomienda usar [Standard Load Balancer](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md).  

En la lista siguiente se muestra la configuración del equilibrador de carga (A)SCS y ERS para este ejemplo de clúster de varios SID con tres sistemas SAP. Necesitará una dirección IP de front-end, sondeos de estado y reglas de equilibrio de carga para cada instancia de ASCS y ERS en cada uno de los SID. Asigne todas las máquinas virtuales que formen parte del clúster de ASCS/ASCS a un grupo de back-end.  

### <a name="ascs"></a>(A)SCS

* Configuración de front-end
  * Dirección IP de NW1:  10.3.1.14
  * Dirección IP de NW2:  10.3.1.16
  * Dirección IP de NW3:  10.3.1.13
* Puertos de sondeo
  * Puerto 620<strong>&lt;nr&gt;</strong>, por lo tanto, los puertos de sondeo 620**00**, 620**10** y 620**20** para NW1, NW2 y NW3
* Reglas de equilibrio de carga: 
* cree una para cada instancia, es decir, NW1/ASCS, NW2/ASCS y NW3/ASCS.
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
  * Dirección IP de NW1: 10.3.1.15
  * Dirección IP de NW2: 10.3.1.17
  * Dirección IP de NW3: 10.3.1.19
* Puerto de sondeo
  * Puerto 621<strong>&lt;nr&gt;</strong>, por lo tanto, los puertos de sondeo 621**02**, 621**12** and 621**22** para NW1, NW2 y NW#
* Reglas de equilibrio de carga: cree una para cada instancia, es decir, NW1/ERS, NW2/ERS y NW3/ERS.
  * Si usa Standard Load Balancer, seleccione **Puertos HA**
  * Si usa Basic Load Balancer, cree reglas de equilibrio de carga para los puertos siguientes
    * 32<strong>&lt;nr&gt;</strong> TCP
    * 33<strong>&lt;nr&gt;</strong> TCP
    * 5<strong>&lt;nr&gt;</strong>13 TCP
    * 5<strong>&lt;nr&gt;</strong>14 TCP
    * 5<strong>&lt;nr&gt;</strong>16 TCP

* Configuración de back-end
  * Se conecta a interfaces de red principales de todas las máquinas que deben ser parte del clúster (A)SCS/ERS


> [!Note]
> Cuando las máquinas virtuales sin direcciones IP públicas se colocan en el grupo de back-end de Standard Load Balancer interno (sin dirección IP pública), no hay conectividad saliente de Internet, a menos que se realice una configuración adicional para permitir el enrutamiento a puntos de conexión públicos. Para obtener más información sobre cómo obtener conectividad saliente, vea [Conectividad de punto de conexión público para máquinas virtuales con Azure Standard Load Balancer en escenarios de alta disponibilidad de SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

> [!IMPORTANT]
> No habilite las marcas de tiempo TCP en VM de Azure que se encuentren detrás de Azure Load Balancer. Si habilita las marcas de tiempo TCP provocará un error en los sondeos de estado. Establezca el parámetro **net.ipv4.tcp_timestamps** a **0**. Consulte [Sondeos de estado de Load Balancer](../../../load-balancer/load-balancer-custom-probe-overview.md) para obtener más información.

## <a name="sap-nfs-shares"></a>Recursos compartidos NFS de SAP

SAP NetWeaver requiere un almacenamiento compartido para el transporte, el directorio de perfil, etc. En el caso de sistemas SAP de alta disponibilidad, es importante tener recursos compartidos NFS de alta disponibilidad. Tendrá que decidir sobre la arquitectura de sus recursos compartidos NFS de SAP. Una opción consiste en compilar un [clúster NFS de alta disponibilidad en máquinas virtuales de Azure en SUSE Linux Enterprise Server][nfs-ha], que se puede compartir entre varios sistemas SAP. 

Otra opción consiste en implementar los recursos compartidos en [volúmenes NFS de Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).  Con Azure NetApp Files, obtendrá alta disponibilidad integrada para los recursos compartidos NFS de SAP.

## <a name="deploy-the-first-sap-system-in-the-cluster"></a>Implementación del primer sistema SAP en el clúster

Ahora que ha decidido la arquitectura de los recursos compartidos NFS de SAP, implemente el primer sistema SAP del clúster, siguiendo la documentación correspondiente.

* Si usa un servidor NFS de alta disponibilidad, siga [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications](./high-availability-guide-suse.md).  
* Si se usa un servidor NFS de alta disponibilidad, siga [Alta disponibilidad de SAP NetWeaver en VM de Azure en SUSE Linux Enterprise Server con Azure NetApp Files para SAP Applications](./high-availability-guide-suse-netapp-files.md).

Los documentos mostrados anteriormente le guiarán por los pasos de preparación de las infraestructuras necesarias, La compilación del clúster y la preparación del sistema operativo para ejecutar la aplicación de SAP.  

> [!TIP]
> Pruebe siempre la funcionalidad de conmutación por error del clúster, una vez implementado el primer sistema, antes de agregar los SID de SAP adicionales al clúster. De este modo sabrá que la funcionalidad del clúster funciona, antes de agregar la complejidad de los sistemas SAP adicionales al clúster.   

## <a name="deploy-additional-sap-systems-in-the-cluster"></a>Implementación de sistemas SAP adicionales en el clúster

En este ejemplo se supone que el sistema **NW1** ya se ha implementado en el clúster. Le mostraremos cómo realizar la implementación en los sistemas SAP de clúster **NW2** y **NW3**. 

Los elementos siguientes tienen el prefijo **[A]** : aplicable a todos los nodos, **[1]** : aplicable solo al nodo 1 o **[2]** : aplicable solo al nodo 2.

### <a name="prerequisites"></a>Prerrequisitos 

> [!IMPORTANT]
> Antes de seguir las instrucciones para implementar sistemas SAP adicionales en el clúster, siga las instrucciones para implementar el primer sistema SAP en el clúster, ya que hay pasos que solo son necesarios durante la implementación del primer sistema.  

En esta documentación se supone que:
* El clúster de Pacemaker ya está configurado y en ejecución.  
* Al menos un sistema SAP (instancia de ASCS/ERS) ya está implementado y está en ejecución en el clúster.  
* Se ha probado la funcionalidad de conmutación por error de clúster.  
* Se implementan los recursos compartidos NFS para todos los sistemas SAP.  

### <a name="prepare-for-sap-netweaver-installation"></a>Preparación de la instalación de SAP NetWeaver

1. Agregue la configuración del sistema recién implementado (es decir, **NW2**, **NW3**) a la instancia existente de Azure Load Balancer, siguiendo las instrucciones de [implementación de Azure Load Balancer manualmente a través de Azure Portal](./high-availability-guide-suse-netapp-files.md#deploy-azure-load-balancer-manually-via-azure-portal). Ajuste las direcciones IP, los puertos de sondeo de estado y las reglas de equilibrio de carga para la configuración.  

2. **[A]** Configure la resolución de nombres para los sistemas SAP adicionales. Puede usar un servidor DNS o modificar `/etc/hosts` en todos los nodos. En este ejemplo se muestra cómo utilizar el archivo `/etc/hosts`.  Adapte las direcciones IP y los nombres de host a su entorno. 

    ```
    sudo vi /etc/hosts
    # IP address of the load balancer frontend configuration for NW2 ASCS
    10.3.1.16 msnw2ascs
    # IP address of the load balancer frontend configuration for NW3 ASCS
    10.3.1.13 msnw3ascs
    # IP address of the load balancer frontend configuration for NW2 ERS
    10.3.1.17 msnw2ers
    # IP address of the load balancer frontend configuration for NW3 ERS
    10.3.1.19 msnw3ers
    # IP address for virtual host name for the NFS server for NW2
    10.3.1.31 nw2-nfs
    # IP address for virtual host name for the NFS server for NW3
    10.3.1.32 nw3-nfs
   ```

3. **[A]** Cree los directorios compartidos para los sistemas SAP adicionales **NW2** y **NW3** que va a implementar en el clúster. 

    ```
    sudo mkdir -p /sapmnt/NW2
    sudo mkdir -p /usr/sap/NW2/SYS
    sudo mkdir -p /usr/sap/NW2/ASCS10
    sudo mkdir -p /usr/sap/NW2/ERS12
    sudo mkdir -p /sapmnt/NW3
    sudo mkdir -p /usr/sap/NW3/SYS
    sudo mkdir -p /usr/sap/NW3/ASCS20
    sudo mkdir -p /usr/sap/NW3/ERS22

    
    sudo chattr +i /sapmnt/NW2
    sudo chattr +i /usr/sap/NW2/SYS
    sudo chattr +i /usr/sap/NW2/ASCS10
    sudo chattr +i /usr/sap/NW2/ERS12
    sudo chattr +i /sapmnt/NW3
    sudo chattr +i /usr/sap/NW3/SYS
    sudo chattr +i /usr/sap/NW3/ASCS20
    sudo chattr +i /usr/sap/NW3/ERS22
   ```

4. **[A]** Configurar `autofs` para montar los sistemas de archivos /sapmnt/SID y /usr/sap/SID/SYS para los sistemas SAP adicionales que va a implementar en el clúster. En este ejemplo, **NW2** y **NW3**.  

   Actualice el objeto `/etc/auto.direct` del archivo con los sistemas de archivos de los sistemas SAP adicionales que va a implementar en el clúster.  

   * Si usa el servidor de archivos NFS, siga las instrucciones incluidas [aquí](./high-availability-guide-suse.md#prepare-for-sap-netweaver-installation).
   * Si usa Azure NetApp Files, siga las instrucciones incluidas [aquí](./high-availability-guide-suse-netapp-files.md#prepare-for-sap-netweaver-installation). 

   Tendrá que reiniciar el servicio de `autofs` para montar los recursos compartidos recién agregados.  

### <a name="install-ascs--ers"></a>Instalación de ASCS/ERS

1. Cree la IP virtual y los recursos del clúster de sondeo para la instancia de ASCS del sistema SAP adicional que va a implementar en el clúster. El ejemplo que se muestra aquí corresponde a ASCS **NW2** y **NW3**, con un servidor NFS de alta disponibilidad.  

   > [!IMPORTANT]
   > Pruebas recientes han mostrado situaciones en las que netcat deja de responder a las solicitudes debido al trabajo pendiente y a su limitación para controlar solo una conexión. El recurso netcat deja de escuchar las solicitudes del equilibrador de carga de Azure y la dirección IP flotante deja de estar disponible.  
   > En el caso de los clústeres de Pacemaker existentes, en el pasado se recomendaba reemplazar netcat por socat. Actualmente se recomienda usar el agente de recursos azure-lb, que forma parte de los agentes de recursos de paquetes, con los siguientes requisitos de versión de paquete:
   > - En el caso de SLES 12 SP4/SP5, la versión debe ser, al menos, resource-agents-4.3.018.a7fb5035-3.30.1.  
   > - Para SLES 15/15 SP1, la versión debe ser al menos resource-agents-4.3.0184.6ee15eb2-4.13.1.  
   >
   > Tenga en cuenta que el cambio requerirá un breve tiempo de inactividad.  
   > En el caso de los clústeres de Pacemaker existentes, si la configuración ya se ha cambiado para usar socat, como se describe en [Protección de la detección del equilibrador de carga de Azure](https://www.suse.com/support/kb/doc/?id=7024128), no hay ningún requisito para cambiar inmediatamente al agente de recursos azure-lb.

    ```
      sudo crm configure primitive fs_NW2_ASCS Filesystem device='nw2-nfs:/NW2/ASCS' directory='/usr/sap/NW2/ASCS10' fstype='nfs4' \
       op start timeout=60s interval=0 \
       op stop timeout=60s interval=0 \
       op monitor interval=20s timeout=40s
   
      sudo crm configure primitive vip_NW2_ASCS IPaddr2 \
        params ip=10.3.1.16 cidr_netmask=24 \
        op monitor interval=10 timeout=20
   
      sudo crm configure primitive nc_NW2_ASCS azure-lb port=62010
   
      sudo crm configure group g-NW2_ASCS fs_NW2_ASCS nc_NW2_ASCS vip_NW2_ASCS \
         meta resource-stickiness=3000

      sudo crm configure primitive fs_NW3_ASCS Filesystem device='nw3-nfs:/NW3/ASCS' directory='/usr/sap/NW3/ASCS20' fstype='nfs4' \
        op start timeout=60s interval=0 \
        op stop timeout=60s interval=0 \
        op monitor interval=20s timeout=40s
   
      sudo crm configure primitive vip_NW3_ASCS IPaddr2 \
       params ip=10.3.1.13 cidr_netmask=24 \
       op monitor interval=10 timeout=20
   
      sudo crm configure primitive nc_NW3_ASCS azure-lb port=62020
   
      sudo crm configure group g-NW3_ASCS fs_NW3_ASCS nc_NW3_ASCS vip_NW3_ASCS \
        meta resource-stickiness=3000
    ```

   A medida que se crean, los recursos pueden asignarse a otros recursos de clúster. Al agruparlos, se migrarán a uno de los nodos del clúster. Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.

2. **[1]** Instale SAP NetWeaver ASCS  

   Instale SAP NetWeaver ASCS como raíz con un nombre de host virtual que se asigna a la dirección IP de la configuración de front-end del equilibrador de carga para ASCS. Por ejemplo, para el sistema **NW2**, el nombre de host virtual es <b>msnw2ascs</b>, <b>10.3.1.16</b> y el número de instancia que usó para el sondeo del equilibrador de carga, por ejemplo <b>10</b>. Para el sistema **NW3**, el nombre de host virtual es <b>msnw3ascs</b>, <b>10.3.1.13</b> y el número de instancia que usó para el sondeo del equilibrador de carga, por ejemplo <b>20</b>.

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst. Puede usar el parámetro SAPINST_USE_HOSTNAME para instalar SAP con el nombre de host virtual.  

     ```
      sudo swpm/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=virtual_hostname
     ```

   Si se produce un error en la instalación para crear una subcarpeta en /usr/sap/**SID**/ASCS**Instance#** , pruebe a establecer el propietario en **sid**adm y el grupo en sapsys de la ASCS**Instance#** e inténtelo de nuevo.

3. **[1]** Cree la IP virtual y los recursos del clúster de sondeo para la instancia de ERS del sistema SAP adicional que va a implementar en el clúster. El ejemplo que se muestra aquí corresponde a ERS **NW2** y **NW3**, con un servidor NFS de alta disponibilidad. 

   ```
    sudo crm configure primitive fs_NW2_ERS Filesystem device='nw2-nfs:/NW2/ASCSERS' directory='/usr/sap/NW2/ERS12' fstype='nfs4' \
      op start timeout=60s interval=0 \
      op stop timeout=60s interval=0 \
      op monitor interval=20s timeout=40s
   
    sudo crm configure primitive vip_NW2_ERS IPaddr2 \
      params ip=10.3.1.17 cidr_netmask=24 \
      op monitor interval=10 timeout=20
   
    sudo crm configure primitive nc_NW2_ERS azure-lb port=62112
   
    sudo crm configure group g-NW2_ERS fs_NW2_ERS nc_NW2_ERS vip_NW2_ERS

    sudo crm configure primitive fs_NW3_ERS Filesystem device='nw3-nfs:/NW3/ASCSERS' directory='/usr/sap/NW3/ERS22' fstype='nfs4' \
      op start timeout=60s interval=0 \
      op stop timeout=60s interval=0 \
      op monitor interval=20s timeout=40s
   
    sudo crm configure primitive vip_NW3_ERS IPaddr2 \
      params ip=10.3.1.19 cidr_netmask=24 \
      op monitor interval=10 timeout=20
   
    sudo crm configure primitive nc_NW3_ERS azure-lb port=62122
   
    sudo crm configure group g-NW3_ERS fs_NW3_ERS nc_NW3_ERS vip_NW3_ERS
   ```

   A medida que se crean los recursos, se les puede asignar a distintos nodos de clúster. Al agruparlos, se migrarán a uno de los nodos del clúster. Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos.  

   A continuación, asegúrese de que los recursos del grupo ERS recién creado se ejecutan en el nodo de clúster, en lugar del nodo de clúster en el que se instaló la instancia de ASCS para el mismo sistema SAP.  Por ejemplo, si NW2 ASCS se instaló en `slesmsscl1`, asegúrese de que el grupo ERS de NW2 se encuentra en ejecución en `slesmsscl2`.  Para migrar el grupo ERS de NW2 a `slesmsscl2`, ejecute el siguiente comando: 

    ```
      crm resource migrate g-NW2_ERS slesmsscl2 force
    ```

4. **[2]** Instale SAP NetWeaver ERS

   Instale SAP NetWeaver ERS como raíz en el otro nodo con un nombre de host virtual que se asigna a la dirección IP de la configuración de front-end del equilibrador de carga para ERS. Por ejemplo, para el sistema **NW2**, el nombre de host virtual es <b>msnw2ers</b>, <b>10.3.1.17</b> y el número de instancia que usó para el sondeo del equilibrador de carga, por ejemplo <b>12</b>. En el caso del sistema **NW3**, el nombre de host virtual es <b>msnw3ers</b>, <b>10.3.1.19</b> y el número de instancia que usó para el sondeo del equilibrador de carga, por ejemplo <b>22</b>. 

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst. Puede usar el parámetro SAPINST_USE_HOSTNAME para instalar SAP con el nombre de host virtual.  

    ```
     sudo swpm/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=virtual_hostname
    ```

   > [!NOTE]
   > Use SWPM SP 20 PL 05 o superior. Las versiones inferiores no establecen correctamente los permisos y se producirá un error de instalación.

   Si se produce un error en la instalación para crear una subcarpeta en /usr/sap/**NW2**/ERS**Instance#** , pruebe a establecer el propietario en **sid**adm y el grupo en sapsys de la ERS**Instance#** e inténtelo de nuevo.

   Si fuera necesario migrar el grupo ERS del sistema SAP recién implementado a otro nodo de clúster, no olvide quitar la restricción de ubicación para el grupo de ERS. Puede quitar la restricción ejecutando el comando siguiente (el ejemplo se da para los sistemas SAP **NW2** y **NW3**).  

    ```
      crm resource unmigrate g-NW2_ERS
      crm resource unmigrate g-NW3_ERS
    ```

5. **[1]** Adapte los perfiles de instancia de ASCS/SCS y ERS para los sistemas SAP recién instalados. El ejemplo que se muestra a continuación corresponde a NW2. Tendrá que adaptar los perfiles ASCS/SCS y ERS para todas las instancias de SAP que se agreguen al clúster.  
 
 * Perfil ASCS/SCS

   ```
   sudo vi /sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs
   
   # Change the restart command to a start command
   #Restart_Program_01 = local $(_EN) pf=$(_PF)
   Start_Program_01 = local $(_EN) pf=$(_PF)
   
   # Add the following lines
   service/halib = $(DIR_CT_RUN)/saphascriptco.so
   service/halib_cluster_connector = /usr/bin/sap_suse_cluster_connector
   
   # Add the keep alive parameter, if using ENSA1
   enque/encni/set_so_keepalive = true
   ```

   En el caso de ENSA1 y ENSA2, asegúrese de que los parámetros del sistema operativo `keepalive` se establecen tal y como se describe en la nota de SAP [1410736](https://launchpad.support.sap.com/#/notes/1410736).  

 * Perfil ERS

   ```
   sudo vi /sapmnt/NW2/profile/NW2_ERS12_msnw2ers
   
   # Change the restart command to a start command
   #Restart_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   Start_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   
   # Add the following lines
   service/halib = $(DIR_CT_RUN)/saphascriptco.so
   service/halib_cluster_connector = /usr/bin/sap_suse_cluster_connector
   
   # remove Autostart from ERS profile
   # Autostart = 1
   ```

6. **[A]** Configure los usuarios de SAP para el sistema SAP recién implementado, en este ejemplo, **NW2** y **NW3**. 

   ```
   # Add sidadm to the haclient group
   sudo usermod -aG haclient nw2adm
   sudo usermod -aG haclient nw3adm
   ```

7. Agregue los servicios de SAP ASCS y ERS para el sistema SAP recién instalado al archivo `sapservice`. El ejemplo que se muestra a continuación corresponde a los sistemas SAP **NW2** y **NW3**.  

   Agregue la entrada del servicio ASCS al segundo nodo y copie la entrada del servicio ERS al primer nodo. Ejecute los comandos para cada sistema SAP en el nodo, donde se instaló la instancia de ASCS para el sistema SAP.  

    ```
     # Execute the following commands on slesmsscl1,assuming the NW2 ASCS instance was installed on slesmsscl1
     cat /usr/sap/sapservices | grep ASCS10 | sudo ssh slesmsscl2 "cat >>/usr/sap/sapservices"
     sudo ssh slesmsscl2 "cat /usr/sap/sapservices" | grep ERS12 | sudo tee -a /usr/sap/sapservices
     # Execute the following commands on slesmsscl2, assuming the NW3 ASCS instance was installed on slesmsscl2
     cat /usr/sap/sapservices | grep ASCS20 | sudo ssh slesmsscl1 "cat >>/usr/sap/sapservices"
     sudo ssh slesmsscl1 "cat /usr/sap/sapservices" | grep ERS22 | sudo tee -a /usr/sap/sapservices
    ```

8. **[1]** Cree los recursos de clúster de SAP para el sistema SAP recién instalado. 

   Si usa la arquitectura de servidor 1 de puesta en cola (ENSA1), defina los recursos para los sistemas SAP **NW2** y **NW3** tal como se indica:

    ```
     sudo crm configure property maintenance-mode="true"
    
     sudo crm configure primitive rsc_sap_NW2_ASCS10 SAPInstance \
      operations \$id=rsc_sap_NW2_ASCS10-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW2_ASCS10_msnw2ascs START_PROFILE="/sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs" \
      AUTOMATIC_RECOVER=false \
      meta resource-stickiness=5000 failure-timeout=60 migration-threshold=1 priority=10
    
     sudo crm configure primitive rsc_sap_NW2_ERS12 SAPInstance \
      operations \$id=rsc_sap_NW2_ERS12-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW2_ERS12_msnw2ers START_PROFILE="/sapmnt/NW2/profile/NW2_ERS12_msnw2ers" AUTOMATIC_RECOVER=false IS_ERS=true \
      meta priority=1000
    
     sudo crm configure modgroup g-NW2_ASCS add rsc_sap_NW2_ASCS10
     sudo crm configure modgroup g-NW2_ERS add rsc_sap_NW2_ERS12
    
     sudo crm configure colocation col_sap_NW2_no_both -5000: g-NW2_ERS g-NW2_ASCS
     sudo crm configure location loc_sap_NW2_failover_to_ers rsc_sap_NW2_ASCS10 rule 2000: runs_ers_NW2 eq 1
     sudo crm configure order ord_sap_NW2_first_start_ascs Optional: rsc_sap_NW2_ASCS10:start rsc_sap_NW2_ERS12:stop symmetrical=false
   
     sudo crm configure primitive rsc_sap_NW3_ASCS20 SAPInstance \
      operations \$id=rsc_sap_NW3_ASCS20-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW3_ASCS10_msnw3ascs START_PROFILE="/sapmnt/NW3/profile/NW3_ASCS20_msnw3ascs" \
      AUTOMATIC_RECOVER=false \
      meta resource-stickiness=5000 failure-timeout=60 migration-threshold=1 priority=10
    
     sudo crm configure primitive rsc_sap_NW3_ERS22 SAPInstance \
      operations \$id=rsc_sap_NW3_ERS22-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW3_ERS22_msnw3ers START_PROFILE="/sapmnt/NW3/profile/NW3_ERS22_msnw2ers" AUTOMATIC_RECOVER=false IS_ERS=true \
      meta priority=1000
    
     sudo crm configure modgroup g-NW3_ASCS add rsc_sap_NW3_ASCS20
     sudo crm configure modgroup g-NW3_ERS add rsc_sap_NW3_ERS22
    
     sudo crm configure colocation col_sap_NW3_no_both -5000: g-NW3_ERS g-NW3_ASCS
     sudo crm configure location loc_sap_NW3_failover_to_ers rsc_sap_NW3_ASCS10 rule 2000: runs_ers_NW3 eq 1
     sudo crm configure order ord_sap_NW3_first_start_ascs Optional: rsc_sap_NW3_ASCS20:start rsc_sap_NW3_ERS22:stop symmetrical=false
     sudo crm configure property maintenance-mode="false"
    ```

   SAP introdujo una opción de compatibilidad con el servidor 2 de puesta en cola, incluida la replicación, a partir de la versión de SAP NW 7.52. A partir de la plataforma ABAP (versión 1809), el servidor 2 de puesta en cola está instalado de forma predeterminada. Consulte la nota de SAP [2630416](https://launchpad.support.sap.com/#/notes/2630416) para consultar la compatibilidad con el servidor 2 de puesta en cola.
   Si usa la arquitectura de servidor 2 de puesta en cola ([ENSA2](https://help.sap.com/viewer/cff8531bc1d9416d91bb6781e628d4e0/1709%20001/en-US/6d655c383abf4c129b0e5c8683e7ecd8.html)), defina los recursos para los sistemas SAP **NW2** y **NW3** tal como se indica:

    ```
     sudo crm configure property maintenance-mode="true"
    
     sudo crm configure primitive rsc_sap_NW2_ASCS10 SAPInstance \
      operations \$id=rsc_sap_NW2_ASCS10-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW2_ASCS10_msnw2ascs START_PROFILE="/sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs" \
      AUTOMATIC_RECOVER=false \
      meta resource-stickiness=5000 
    
     sudo crm configure primitive rsc_sap_NW2_ERS12 SAPInstance \
      operations \$id=rsc_sap_NW2_ERS12-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW2_ERS12_msnw2ers START_PROFILE="/sapmnt/NW2/profile/NW2_ERS12_msnw2ers" AUTOMATIC_RECOVER=false IS_ERS=true 
    
     sudo crm configure modgroup g-NW2_ASCS add rsc_sap_NW2_ASCS10
     sudo crm configure modgroup g-NW2_ERS add rsc_sap_NW2_ERS12
    
     sudo crm configure colocation col_sap_NW2_no_both -5000: g-NW2_ERS g-NW2_ASCS
     sudo crm configure order ord_sap_NW2_first_start_ascs Optional: rsc_sap_NW2_ASCS10:start rsc_sap_NW2_ERS12:stop symmetrical=false
   
     sudo crm configure primitive rsc_sap_NW3_ASCS20 SAPInstance \
      operations \$id=rsc_sap_NW3_ASCS20-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW3_ASCS10_msnw3ascs START_PROFILE="/sapmnt/NW3/profile/NW3_ASCS20_msnw3ascs" \
      AUTOMATIC_RECOVER=false \
      meta resource-stickiness=5000
    
     sudo crm configure primitive rsc_sap_NW3_ERS22 SAPInstance \
      operations \$id=rsc_sap_NW3_ERS22-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW3_ERS22_msnw3ers START_PROFILE="/sapmnt/NW3/profile/NW3_ERS22_msnw2ers" AUTOMATIC_RECOVER=false IS_ERS=true
    
     sudo crm configure modgroup g-NW3_ASCS add rsc_sap_NW3_ASCS20
     sudo crm configure modgroup g-NW3_ERS add rsc_sap_NW3_ERS22
    
     sudo crm configure colocation col_sap_NW3_no_both -5000: g-NW3_ERS g-NW3_ASCS
     sudo crm configure order ord_sap_NW3_first_start_ascs Optional: rsc_sap_NW3_ASCS20:start rsc_sap_NW3_ERS22:stop symmetrical=false
     sudo crm configure property maintenance-mode="false"
    ```

   Si está actualizando desde una versión anterior y va a cambiar al servidor 2 de puesta en cola, consulte la nota de SAP [2641019](https://launchpad.support.sap.com/#/notes/2641019). 

   Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.
   En el ejemplo siguiente se muestra el estado de los recursos de clúster después de haber agregado los sistemas SAP **NW2** y **NW3** al clúster. 

    ```
     sudo crm_mon -r
    
    # Online: [ slesmsscl1 slesmsscl2 ]
    
    #Full list of resources:
    
    #stonith-sbd     (stonith:external/sbd): Started slesmsscl1
    # Resource Group: g-NW1_ASCS
    #     fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
    #     nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
    #     vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
    #     rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
    # Resource Group: g-NW1_ERS
    #     fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
    #     nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
    #     vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
    #     rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
    # Resource Group: g-NW2_ASCS
    #     fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
    #     nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
    #     vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
    #     rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
    # Resource Group: g-NW2_ERS
    #     fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
    #     nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
    #     vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
    #     rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
    # Resource Group: g-NW3_ASCS
    #     fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
    #     nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
    #     vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
    #     rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
    # Resource Group: g-NW3_ERS
    #     fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
    #     nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
    #     vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
    #     rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
    ```

   En la imagen siguiente se muestra el aspecto que tendrían los recursos en la Web Konsole de alta disponibilidad (Hawk), con los recursos del sistema SAP **NW2** expandido.  

   [![Información general sobre la alta disponibilidad de SAP NetWeaver](./media/high-availability-guide-suse/ha-suse-multi-sid-hawk.png)](./media/high-availability-guide-suse/ha-suse-multi-sid-hawk-detail.png#lightbox)

### <a name="proceed-with-the-sap-installation"></a>Continuación de la instalación de SAP 

Complete la instalación de SAP mediante las siguientes tareas:

* [Preparación de los servidores de aplicaciones de SAP NetWeaver](./high-availability-guide-suse.md#2d6008b0-685d-426c-b59e-6cd281fd45d7)
* [Instalación de una instancia de DBMS](./high-availability-guide-suse.md#install-database)
* [Instalación de un servidor de aplicaciones de SAP principal](./high-availability-guide-suse.md#sap-netweaver-application-server-installation)
* Instalación de una o varias instancias de aplicación de SAP adicionales

## <a name="test-the-multi-sid-cluster-setup"></a>Prueba de la configuración del clúster de varios SID

Las siguientes pruebas constituyen un subconjunto de los casos de prueba de las guías de procedimientos recomendados de SUSE. Se incluyen en este artículo para su mayor comodidad. Para obtener una lista completa de las pruebas del clúster, consulte la siguiente documentación:

* Si usa un servidor NFS de alta disponibilidad, siga [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications](./high-availability-guide-suse.md).  
* Si se usa un servidor NFS de alta disponibilidad, siga [Alta disponibilidad de SAP NetWeaver en VM de Azure en SUSE Linux Enterprise Server con Azure NetApp Files para SAP Applications](./high-availability-guide-suse-netapp-files.md).

Además, lea siempre las guías de procedimientos recomendados y realice todas las pruebas adicionales que puedan haberse agregado.  
Las pruebas que se presentan se encuentran en un clúster de dos nodos y varios SID con tres sistemas SAP instalados.  

1. Prueba de HAGetFailoverConfig y HACheckFailoverConfig

   Ejecute los siguientes comandos como <sapsid>adm en el nodo donde se ejecuta actualmente la instancia de ASCS. Si los comandos producen un error de memoria insuficiente, los guiones en el nombre de host pueden ser la causa. Se trata de un problema conocido que SUSE corregirá en el paquete sap-suse-cluster-connector.

   ```
    slesmsscl1:nw1adm 57> sapcontrol -nr 00 -function HAGetFailoverConfig

   # 10.12.2019 21:33:08
   # HAGetFailoverConfig
   # OK
   # HAActive: TRUE
   # HAProductVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4
   # HASAPInterfaceVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4 (sap_suse_cluster_connector 3.1.0)
   # HADocumentation: https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
   # HAActiveNode: slesmsscl1
   # HANodes: slesmsscl1, slesmsscl2

    slesmsscl1:nw1adm 53> sapcontrol -nr 00 -function HACheckFailoverConfig

    # 19.12.2019 21:19:58
    # HACheckFailoverConfig
    # OK
    # state, category, description, comment
    # SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version, SAPInstance includes is-ers patch

    slesmsscl2:nw2adm 35> sapcontrol -nr 10 -function HAGetFailoverConfig

   # 10.12.2019 21:37:09
   # HAGetFailoverConfig
   # OK
   # HAActive: TRUE
   # HAProductVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4
   # HASAPInterfaceVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4 (sap_suse_cluster_connector 3.1.0)
   # HADocumentation: https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
   # HAActiveNode: slesmsscl2
   # HANodes: slesmsscl2, slesmsscl1

    slesmsscl2:nw2adm 52> sapcontrol -nr 10 -function HACheckFailoverConfig

    # 19.12.2019 21:17:39
    # HACheckFailoverConfig
    # OK
    # state, category, description, comment
    # SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version, SAPInstance includes is-ers patch

    slesmsscl1:nw3adm 49> sapcontrol -nr 20 -function HAGetFailoverConfig

   # 10.12.2019 23:35:36
   # HAGetFailoverConfig
   # OK
   # HAActive: TRUE
   # HAProductVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4
   # HASAPInterfaceVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4 (sap_suse_cluster_connector 3.1.0)
   # HADocumentation: https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
   # HAActiveNode: slesmsscl1
   # HANodes: slesmsscl1, slesmsscl2

    slesmsscl1:nw3adm 52> sapcontrol -nr 20 -function HACheckFailoverConfig

    # 19.12.2019 21:10:42
    # HACheckFailoverConfig
    # OK
    # state, category, description, comment
    # SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version, SAPInstance includes is-ers patch
   ```

2. Migración manual de la instancia de ASCS. En el ejemplo se muestra cómo migrar la instancia de ASCS para SAP System NW2.  
   Estado del recurso antes de iniciar la prueba:
   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

   Ejecute los siguientes comandos como raíz para migrar la instancia de NW2 ASCS.

   ```
    crm resource migrate rsc_sap_NW2_ASCS10 force
    # INFO: Move constraint created for rsc_sap_NW2_ASCS10
    
    crm resource unmigrate rsc_sap_NW2_ASCS10
   # INFO: Removed migration constraints for rsc_sap_NW2_ASCS10
   
   # Remove failed actions for the ERS that occurred as part of the migration
    crm resource cleanup rsc_sap_NW2_ERS12
   ```

   Estado del recurso después de la prueba:

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

1. Pruebe HAFailoverToNode. En la prueba presentada aquí se muestra cómo migrar la instancia de ASCS para SAP System NW2.  

   Estado del recurso antes de iniciar la prueba:

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

   Ejecute los siguientes comandos como **nw2** para migrar la instancia de NW2 ASCS.

   ```
    slesmsscl2:nw2adm 53> sapcontrol -nr 10 -host msnw2ascs -user nw2adm password -function HAFailoverToNode ""
   
   # run as root
   # Remove failed actions for the ERS that occurred as part of the migration
   crm resource cleanup rsc_sap_NW2_ERS12
   # Remove migration constraints
   crm resource clear rsc_sap_NW2_ASCS10
   #INFO: Removed migration constraints for rsc_sap_NW2_ASCS10
   ```

   Estado del recurso después de la prueba:

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

1. Simulación de bloqueo de nodo

   Estado del recurso antes de iniciar la prueba:

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

   Ejecute el siguiente comando como raíz en el nodo en el que haya al menos una instancia de ASCS en ejecución. En este ejemplo, se ejecutó el comando en `slesmsscl2`, donde las instancias de ASCS para NW1 y NW3 están en ejecución.  

   ```
    slesmsscl2:~ # echo b > /proc/sysrq-trigger
   ```

   Si usa SBD, Pacemaker no se iniciará automáticamente en el nodo terminado. El estado después de que el nodo se inicia de nuevo debe parecerse a este.

   ```
    Online: [ slesmsscl1 ]
    OFFLINE: [ slesmsscl2 ]
    Full list of resources:

    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
    
    Failed Resource Actions:
    * rsc_sap_NW1_ERS02_monitor_11000 on slesmsscl1 'not running' (7): call=125, status=complete, exitreason='',
        last-rc-change='Fri Dec 13 19:32:10 2019', queued=0ms, exec=0ms
    * rsc_sap_NW2_ERS12_monitor_11000 on slesmsscl1 'not running' (7): call=126, status=complete, exitreason='',
        last-rc-change='Fri Dec 13 19:32:10 2019', queued=0ms, exec=0ms
    * rsc_sap_NW3_ERS22_monitor_11000 on slesmsscl1 'not running' (7): call=127, status=complete, exitreason='',
        last-rc-change='Fri Dec 13 19:32:10 2019', queued=0ms, exec=0ms
   ```

   Use los siguientes comandos para iniciar Pacemaker en el nodo terminado, limpiar los mensajes de SBD y limpiar los recursos con error.

   ```# run as root
   # list the SBD device(s)
   slesmsscl2:~ # cat /etc/sysconfig/sbd | grep SBD_DEVICE=
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116;/dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1;/dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3"
   
   slesmsscl2:~ # sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116 -d /dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1 -d /dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3 message slesmsscl2 clear
   
   slesmsscl2:~ # systemctl start pacemaker
   slesmsscl2:~ # crm resource cleanup rsc_sap_NW1_ERS02
   slesmsscl2:~ # crm resource cleanup rsc_sap_NW2_ERS12
   slesmsscl2:~ # crm resource cleanup rsc_sap_NW3_ERS22
   ```

   Estado del recurso después de la prueba:

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
   ```

## <a name="next-steps"></a>Pasos siguientes

* [Planeamiento e implementación de Azure Virtual Machines para SAP][planning-guide]
* [Implementación de Azure Virtual Machines para SAP][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP][dbms-guide]
* Para más información sobre cómo establecer la alta disponibilidad y planear la recuperación ante desastres de SAP HANA en Azure Virtual Machines, consulte [Alta disponibilidad de SAP HANA en las máquinas virtuales de Azure][sap-hana-ha].
