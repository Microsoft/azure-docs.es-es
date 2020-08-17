---
title: 'Alta disponibilidad de las VM de Azure para SAP NW en RHEL: guía de varios SID | Microsoft Docs'
description: Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver en Red Hat Enterprise Linux
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/04/2020
ms.author: radeltch
ms.openlocfilehash: 892c45db835457d5f0127d7377d722fc7f0df518
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87760760"
---
# <a name="high-availability-for-sap-netweaver-on-azure-vms-on-red-hat-enterprise-linux-for-sap-applications-multi-sid-guide"></a>Alta disponibilidad para SAP NetWeaver en VM de Azure en Red Hat Enterprise Linux para SAP Applications: guía de varios SID

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-azure-doc]:https://docs.microsoft.com/azure/azure-netapp-files/
[anf-avail-matrix]:https://azure.microsoft.com/global-infrastructure/services/?products=storage&regions=all
[anf-register]:https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register
[anf-sap-applications-azure]:https://www.netapp.com/us/media/tr-4746.pdf

[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2009879]:https://launchpad.support.sap.com/#/notes/2009879
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[1410736]:https://launchpad.support.sap.com/#/notes/1410736

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[template-multisid-xscs]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability-rhel.md
[glusterfs-ha]:high-availability-guide-rhel-glusterfs.md

En este artículo se describe cómo implementar varios sistemas de alta disponibilidad de SAP NetWeaver (es decir, varios SID) en un clúster de dos nodos en VM de Azure con Red Hat Enterprise Linux para SAP Applications.  

En las configuraciones de ejemplo, los comandos de instalación, etc., se implementan tres sistemas SAP NetWeaver 7.50 en un único clúster de alta disponibilidad de dos nodos. Los SID de los sistemas SAP son los siguientes:
* **NW1**: Número de instancia de ASCS **00** y nombre de host virtual **msnw1ascs**; número de instancia de ERS **02** y nombre de host virtual **msnw1ers**.  
* **NW2**: Número de instancia de ASCS **10** y nombre de host virtual **msnw2ascs**; número de instancia de ERS **12** y nombre de host virtual **msnw2ers**.  
* **NW3**: Número de instancia de ASCS **20** y nombre de host virtual **msnw3ascs**; número de instancia de ERS **22** y nombre de host virtual **msnw3ers**.  

El artículo no trata la capa de base de datos ni la implementación de los recursos compartidos NFS de SAP. En los ejemplos de este artículo, usamos el volumen **sapMSID** de [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md) para los recursos compartidos de NFS, suponiendo que el volumen ya está implementado. También suponemos que el volumen de Azure NetApp Files se ha implementado en el protocolo NFSv3 y que las siguientes rutas de acceso existen para los recursos de clúster para las instancias de ASCS y ERS de los sistemas SAP NW1, NW2 y NW3:  

* volumen sapMSID (nfs://10.42.0.4/sapmnt<b>NW1</b>)
* volumen sapMSID (nfs://10.42.0.4/usrsap<b>NW1</b>ascs)
* volumen sapMSID (nfs://10.42.0.4/usrsap<b>NW1</b>sys)
* volumen sapMSID (nfs://10.42.0.4/usrsap<b>NW1</b>ers)
* volumen sapMSID (nfs://10.42.0.4/sapmnt<b>NW2</b>)
* volumen sapMSID (nfs://10.42.0.4/usrsap<b>NW2</b>ascs)
* volumen sapMSID (nfs://10.42.0.4/usrsap<b>NW2</b>sys)
* volumen sapMSID (nfs://10.42.0.4/usrsap<b>NW2</b>ers)
* volumen sapMSID (nfs://10.42.0.4/sapmnt<b>NW3</b>)
* volumen sapMSID (nfs://10.42.0.4/usrsap<b>NW3</b>ascs)
* volumen sapMSID (nfs://10.42.0.4/usrsap<b>NW3</b>sys)
* volumen sapMSID (nfs://10.42.0.4/usrsap<b>NW3</b>ers)

Antes de comenzar, consulte las siguientes notas y documentos de SAP:

* Nota de SAP [1928533], que incluye:
  * La lista de tamaños de máquina virtual de Azure que se admiten para la implementación del software de SAP
  * Información importante sobre la capacidad para los tamaños de máquina virtual de Azure
  * Software de SAP admitido y combinaciones de sistema operativo y base de datos
  * Versión del kernel de SAP requerida para Windows y Linux en Microsoft Azure
* [Documentación sobre Azure NetApp Files][anf-azure-doc]
* La nota de SAP [2015553] enumera los requisitos previos para las implementaciones de software de SAP admitidas por SAP en Azure.
* La nota de SAP [2002167] recomienda la configuración del sistema operativo para Red Hat Enterprise Linux
* La nota de SAP [2009879] contiene las instrucciones de SAP HANA para Red Hat Enterprise Linux
* La nota de SAP [2178632] contiene información detallada sobre todas las métricas de supervisión notificadas para SAP en Azure.
* La nota de SAP [2191498] incluye la versión de SAP Host Agent necesaria para Linux en Azure.
* La nota de SAP [2243692] incluye información acerca de las licencias de SAP en Linux en Azure.
* La nota de SAP [1999351] contiene más soluciones de problemas de la extensión de supervisión mejorada de Azure para SAP.
* La [WIKI de la comunidad SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contiene todas las notas de SAP que se necesitan para Linux.
* [Planeación e implementación de Azure Virtual Machines para SAP en Linux][planning-guide]
* [Implementación de Azure Virtual Machines para SAP en Linux][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP en Linux][dbms-guide]
* [SAP Netweaver en el clúster de Pacemaker](https://access.redhat.com/articles/3150081)
* Documentación general de RHEL
  * [Introducción al complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [Administración del complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [Referencia del complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
  * [Configuración de ASCS/ERS para SAP Netweaver con recursos independientes en RHEL 7.5](https://access.redhat.com/articles/3569681)
  * [Configuración de SAP S/4HANA ASCS/ERS con el servidor 2 de puesta en cola independiente (ENSA2) en Pacemaker en RHEL](https://access.redhat.com/articles/3974941)
* Documentación de RHEL específica para Azure:
  * [Directivas de compatibilidad para clústeres de alta disponibilidad RHEL: instancias de Microsoft Azure Virtual Machines como miembros del clúster](https://access.redhat.com/articles/3131341)
  * [Instalación y configuración de un clúster de alta disponibilidad de Red Hat Enterprise Linux 7.4 (y versiones posteriores) en Microsoft Azure](https://access.redhat.com/articles/3252491)
* [Aplicaciones de NetApp SAP en Microsoft Azure que usan Azure NetApp Files][anf-sap-applications-azure]

## <a name="overview"></a>Información general

Se debe ajustar el tamaño de las máquinas virtuales que participan en el clúster para poder ejecutar todos los recursos en caso de que se produzca la conmutación por error. Cada SID de SAP puede conmutar por error de forma independiente en el clúster de alta disponibilidad de varios SID.  

Para lograr alta disponibilidad, SAP NetWeaver requiere recursos compartidos disponibles. En esta documentación, presentamos los ejemplos con los recursos compartidos de SAP implementados en [volúmenes de Azure NetApp Files NFS](../../../azure-netapp-files/azure-netapp-files-create-volumes.md). También es posible hospedar los recursos compartidos en un [clúster de GlusterFS](./high-availability-guide-rhel-glusterfs.md) de alta disponibilidad, que pueden usar varios sistemas SAP.  

![Información general sobre la alta disponibilidad de SAP NetWeaver](./media/high-availability-guide-rhel/ha-rhel-multi-sid.png)

> [!IMPORTANT]
> La compatibilidad con la agrupación en clústeres de varios SID de SAP ASCS/ERS con Red Hat Linux como sistema operativo invitado en las VM de Azure se limita a **cinco** SID de SAP en un mismo clúster. Cada nuevo SID aumenta la complejidad. No **se admite** una combinación de SAP Enqueue Replication Server 1 y SAP Enqueue Replication Server 2 en el mismo clúster. La agrupación en clústeres de varios SID describe la instalación de varias instancias de SAP ASCS/ERS con SID diferentes en un clúster de Pacemaker. Actualmente, la agrupación en clústeres de varios SID solo se admite para ASCS/ERS.  

> [!TIP]
> La agrupación en clústeres de varios SID de SAP ASCS/ERS es una solución de gran complejidad. Su implementación resulta más compleja. También implica un mayor esfuerzo administrativo al ejecutar actividades de mantenimiento (como la aplicación de revisiones del SO). Antes de comenzar la implementación real, dedique tiempo a planear cuidadosamente la implementación y todos los componentes implicados como máquinas virtuales, montajes NFS, VIP, configuraciones de equilibrador de carga, etc.  

SAP NetWeaver ASCS, SAP NetWeaver SCS y SAP NetWeaver ERS usan direcciones IP virtuales y el nombre de host virtual. En Azure, se requiere un equilibrador de carga para usar una dirección IP virtual. Se recomienda usar [Standard Load Balancer](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md).  

En la lista siguiente se muestra la configuración del equilibrador de carga (A)SCS y ERS para este ejemplo de clúster de varios SID con tres sistemas SAP. Necesitará una dirección IP de front-end, sondeos de estado y reglas de equilibrio de carga para cada instancia de ASCS y ERS en cada uno de los SID. Asigne todas las VM que formen parte del clúster de ASCS/ASCS a un grupo de back-end de un ILB único.  

### <a name="ascs"></a>(A)SCS

* Configuración de front-end
  * Dirección IP de NW1:  10.3.1.50
  * Dirección IP de NW2:  10.3.1.52
  * Dirección IP de NW3:  10.3.1.54

* Puertos de sondeo
  * Puerto 620<strong>&lt;nr&gt;</strong>, por lo tanto, los puertos de sondeo 620**00**, 620**10** y 620**20** para NW1, NW2 y NW3
* Reglas de equilibrio de carga: cree una para cada instancia, es decir, NW1/ASCS, NW2/ASCS y NW3/ASCS.
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
  * Dirección IP de NW1 10.3.1.51
  * Dirección IP de NW2 10.3.1.53
  * Dirección IP de NW3 10.3.1.55

* Puerto de sondeo
  * Puerto 621<strong>&lt;nr&gt;</strong>, por lo tanto, los puertos de sondeo 621**02**, 621**12** y 621**22** para NW1, NW2 y N3
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

## <a name="sap-shares"></a>Recursos compartidos de SAP

SAP NetWeaver requiere un almacenamiento compartido para el transporte, el directorio de perfil, etc. En el caso de sistemas SAP de alta disponibilidad, es importante tener recursos compartidos de alta disponibilidad. Tendrá que decidir sobre la arquitectura de sus recursos compartidos de SAP. Una opción consiste en implementar los recursos compartidos en [Volúmenes NFS de Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).  Con Azure NetApp Files, obtendrá alta disponibilidad integrada para los recursos compartidos NFS de SAP.

Otra opción consiste en compilar [GlusterFS en VM de Azure en Red Hat Enterprise Linux para SAP NetWeaver](./high-availability-guide-rhel-glusterfs.md), que se puede compartir entre varios sistemas SAP. 

## <a name="deploy-the-first-sap-system-in-the-cluster"></a>Implementación del primer sistema SAP en el clúster

Ahora que ha decidido la arquitectura de los recursos compartidos de SAP, implemente el primer sistema SAP del clúster, siguiendo la documentación correspondiente.

* Si se usan volúmenes NFS de Azure NetApp Files, siga [Alta disponibilidad de VM de Azure para SAP NetWeaver en Red Hat Enterprise Linux con Azure NetApp Files para SAP Applications](./high-availability-guide-rhel-netapp-files.md).  
* Si usa el clúster GlusterFS, siga [GlusterFS en VM de Azure de Red Hat Enterprise Linux para SAP NetWeaver](./high-availability-guide-rhel-glusterfs.md).  

Los documentos mostrados anteriormente le guiarán por los pasos de preparación de las infraestructuras necesarias, la compilación del clúster y la preparación del sistema operativo para ejecutar la aplicación de SAP.  

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

1. Agregue la configuración del sistema recién implementado (es decir, **NW2**, **NW3**) a la instancia existente de Azure Load Balancer, siguiendo las instrucciones de [implementación de Azure Load Balancer manualmente a través de Azure Portal](./high-availability-guide-rhel-netapp-files.md#deploy-linux-manually-via-azure-portal). Ajuste las direcciones IP, los puertos de sondeo de estado y las reglas de equilibrio de carga para la configuración.  

2. **[A]** Configure la resolución de nombres para los sistemas SAP adicionales. Puede usar un servidor DNS o modificar `/etc/hosts` en todos los nodos. En este ejemplo se muestra cómo utilizar el archivo `/etc/hosts`.  Adapte las direcciones IP y los nombres de host a su entorno. 

    ```
    sudo vi /etc/hosts
    # IP address of the load balancer frontend configuration for NW2 ASCS
    10.3.1.52 msnw2ascs
    # IP address of the load balancer frontend configuration for NW3 ASCS
    10.3.1.54 msnw3ascs
    # IP address of the load balancer frontend configuration for NW2 ERS
    10.3.1.53 msnw2ers
    # IP address of the load balancer frontend configuration for NW3 ERS
    10.3.1.55 msnw3ers
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

4. **[A]** Agregue las entradas de montaje para los sistemas de archivos /sapmnt/SID y /usr/sap/SID/SYS para los sistemas SAP adicionales que va a implementar en el clúster. En este ejemplo, **NW2** y **NW3**.  

   Actualice el objeto `/etc/fstab` del archivo con los sistemas de archivos de los sistemas SAP adicionales que va a implementar en el clúster.  

   * Si usa Azure NetApp Files, siga las instrucciones incluidas [aquí](./high-availability-guide-rhel-netapp-files.md#prepare-for-sap-netweaver-installation).  
   * Si usa el clúster GlusterFS, siga las instrucciones incluidas [aquí](./high-availability-guide-rhel.md#prepare-for-sap-netweaver-installation).  

### <a name="install-ascs--ers"></a>Instalación de ASCS/ERS

1. Cree la IP virtual y los recursos del clúster de sondeo para la instancia de ASCS del sistema SAP adicional que va a implementar en el clúster. El ejemplo que se muestra aquí es para ASCS de **NW2** y **NW3**, mediante NFS en volúmenes de Azure NetApp Files con el protocolo NFSv3.  

    ```
    sudo pcs resource create fs_NW2_ASCS Filesystem device='10.42.0.4:/sapMSIDR/usrsapNW2ascs' \
    directory='/usr/sap/NW2/ASCS10' fstype='nfs' force_unmount=safe \
    op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-NW2_ASCS

    sudo pcs resource create vip_NW2_ASCS IPaddr2 \
    ip=10.3.1.52 cidr_netmask=24 \
     --group g-NW2_ASCS
    
    sudo pcs resource create nc_NW2_ASCS azure-lb port=62010 \
     --group g-NW2_ASCS

    sudo pcs resource create fs_NW3_ASCS Filesystem device='10.42.0.4:/sapMSIDR/usrsapNW3ascs' \
    directory='/usr/sap/NW3/ASCS20' fstype='nfs' force_unmount=safe \
    op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
    --group g-NW3_ASCS

    sudo pcs resource create vip_NW3_ASCS IPaddr2 \
    ip=10.3.1.54 cidr_netmask=24 \
    --group g-NW3_ASCS

    sudo pcs resource create nc_NW3_ASCS azure-lb port=62020 \
    --group g-NW3_ASCS
    ```

   Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.  

2. **[1]** Instale SAP NetWeaver ASCS  

   Instale SAP NetWeaver ASCS como raíz con un nombre de host virtual que se asigna a la dirección IP de la configuración de front-end del equilibrador de carga para ASCS. Por ejemplo, para el sistema **NW2**, el nombre de host virtual es <b>msnw2ascs</b>, <b>10.3.1.52</b> y el número de instancia que usó para el sondeo del equilibrador de carga, por ejemplo <b>10</b>. Para el sistema **NW3**, el nombre de host virtual es <b>msnw3ascs</b>, <b>10.3.1.54</b> y el número de instancia que usó para el sondeo del equilibrador de carga, por ejemplo <b>20</b>. Anote en qué nodo de clúster instala ASCS para cada SID de SAP.  

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst. Puede usar el parámetro SAPINST_USE_HOSTNAME para instalar SAP con el nombre de host virtual.  

    ```
    # Allow access to SWPM. This rule is not permanent. If you reboot the machine, you have to run the command again
    sudo firewall-cmd --zone=public --add-port=4237/tcp
    sudo swpm/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=virtual_hostname
    ```

   Si se produce un error en la instalación para crear una subcarpeta en /usr/sap/**SID**/ASCS**Instance#** , pruebe a establecer el propietario en **sid**adm y el grupo en sapsys de la ASCS**Instance#** e inténtelo de nuevo.

3. **[1]** Cree la IP virtual y los recursos del clúster de sondeo para la instancia de ERS del sistema SAP adicional que va a implementar en el clúster. El ejemplo que se muestra aquí es para ERS de **NW2** y **NW3**, mediante NFS en volúmenes de Azure NetApp Files con el protocolo NFSv3.  

    ```
    sudo pcs resource create fs_NW2_AERS Filesystem device='10.42.0.4:/sapMSIDR/usrsapNW2ers' \
    directory='/usr/sap/NW2/ERS12' fstype='nfs' force_unmount=safe \
    op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-NW2_AERS

    sudo pcs resource create vip_NW2_AERS IPaddr2 \
    ip=10.3.1.53 cidr_netmask=24 \
     --group g-NW2_AERS

    sudo pcs resource create nc_NW2_AERS azure-lb port=62112 \
     --group g-NW2_AERS

    sudo pcs resource create fs_NW3_AERS Filesystem device='10.42.0.4:/sapMSIDR/usrsapNW3ers' \
    directory='/usr/sap/NW3/ERS22' fstype='nfs' force_unmount=safe \
    op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-NW3_AERS

    sudo pcs resource create vip_NW3_AERS IPaddr2 \
    ip=10.3.1.55 cidr_netmask=24 \
     --group g-NW3_AERS

    sudo pcs resource create nc_NW3_AERS azure-lb port=62122 \
     --group g-NW3_AERS
   ```

   Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos.  

   A continuación, asegúrese de que los recursos del grupo ERS recién creado se ejecutan en el nodo de clúster, en lugar del nodo de clúster en el que se instaló la instancia de ASCS para el mismo sistema SAP.  Por ejemplo, si NW2 ASCS se instaló en `rhelmsscl1`, asegúrese de que el grupo ERS de NW2 se encuentra en ejecución en `rhelmsscl2`.  Para migrar el grupo ERS de NW2 a `rhelmsscl2`, ejecute el siguiente comando para uno de los recursos de clúster del grupo: 

    ```
      pcs resource move fs_NW2_AERS rhelmsscl2
    ```

4. **[2]** Instale SAP NetWeaver ERS

   Instale SAP NetWeaver ERS como raíz en el otro nodo con un nombre de host virtual que se asigna a la dirección IP de la configuración de front-end del equilibrador de carga para ERS. Por ejemplo, para el sistema **NW2**, el nombre de host virtual es <b>msnw2ers</b>, <b>10.3.1.53</b> y el número de instancia que usó para el sondeo del equilibrador de carga, por ejemplo <b>12</b>. En el caso del sistema **NW3**, el nombre de host virtual es <b>msnw3ers</b>, <b>10.3.1.55</b> y el número de instancia que usó para el sondeo del equilibrador de carga, por ejemplo <b>22</b>. 

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst. Puede usar el parámetro SAPINST_USE_HOSTNAME para instalar SAP con el nombre de host virtual.  

    ```
    # Allow access to SWPM. This rule is not permanent. If you reboot the machine, you have to run the command again
    sudo firewall-cmd --zone=public --add-port=4237/tcp
    sudo swpm/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=virtual_hostname
    ```

   > [!NOTE]
   > Use SWPM SP 20 PL 05 o superior. Las versiones inferiores no establecen correctamente los permisos y se producirá un error de instalación.

   Si se produce un error en la instalación para crear una subcarpeta en /usr/sap/**NW2**/ERS**Instance#** , pruebe a establecer el propietario en **sid**adm y el grupo en sapsys de la ERS**Instance#** e inténtelo de nuevo.

   Si fuera necesario migrar el grupo ERS del sistema SAP recién implementado a otro nodo de clúster, no olvide quitar la restricción de ubicación para el grupo de ERS. Puede quitar la restricción ejecutando el comando siguiente (el ejemplo se da para los sistemas SAP **NW2** y **NW3**). Asegúrese de quitar las restricciones temporales del mismo grupo que uso en el comando para mover el grupo de clúster de ERS.

    ```
      pcs resource clear fs_NW2_AERS
      pcs resource clear fs_NW3_AERS
    ```

5. **[1]** Adapte los perfiles de instancia de ASCS/SCS y ERS para los sistemas SAP recién instalados. El ejemplo que se muestra a continuación corresponde a NW2. Tendrá que adaptar los perfiles ASCS/SCS y ERS para todas las instancias de SAP que se agreguen al clúster.  
 
   * Perfil ASCS/SCS

      ```
      sudo vi /sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs
   
      # Change the restart command to a start command
      #Restart_Program_01 = local $(_EN) pf=$(_PF)
      Start_Program_01 = local $(_EN) pf=$(_PF)

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
   
      # remove Autostart from ERS profile
      # Autostart = 1
      ```

6. **[A]** Actualización del archivo /usr/sap/sapservices

   Para evitar que el script de arranque sapinit inicie las instancias, todas las que sean administradas por Pacemaker se deben marcar como comentario en el archivo `/usr/sap/sapservices`.  El ejemplo que se muestra a continuación corresponde a los sistemas SAP **NW2** y **NW3**.  

   ```
    # On the node where ASCS was installed, comment out the line for the ASCS instacnes
    #LD_LIBRARY_PATH=/usr/sap/NW2/ASCS10/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/NW2/ASCS10/exe/sapstartsrv pf=/usr/sap/NW2/SYS/profile/NW2_ASCS10_msnw2ascs -D -u nw2adm
    #LD_LIBRARY_PATH=/usr/sap/NW3/ASCS20/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/NW3/ASCS20/exe/sapstartsrv pf=/usr/sap/NW3/SYS/profile/NW3_ASCS20_msnw3ascs -D -u nw3adm

    # On the node where ERS was installed, comment out the line for the ERS instacnes
    #LD_LIBRARY_PATH=/usr/sap/NW2/ERS12/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/NW2/ERS12/exe/sapstartsrv pf=/usr/sap/NW2/ERS12/profile/NW2_ERS12_msnw2ers -D -u nw2adm
    #LD_LIBRARY_PATH=/usr/sap/NW3/ERS22/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/NW3/ERS22/exe/sapstartsrv pf=/usr/sap/NW3/ERS22/profile/NW3_ERS22_msnw3ers -D -u nw3adm
   ```

7. **[1]** Cree los recursos de clúster de SAP para el sistema SAP recién instalado.  

   Si usa la arquitectura de servidor 1 de puesta en cola (ENSA1), defina los recursos para los sistemas SAP **NW2** y **NW3** tal como se indica:

    ```
     sudo pcs property set maintenance-mode=true

    sudo pcs resource create rsc_sap_NW2_ASCS10 SAPInstance \
    InstanceName=NW2_ASCS10_msnw2ascs START_PROFILE="/sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW2_ASCS

    sudo pcs resource create rsc_sap_NW2_ERS12 SAPInstance \
    InstanceName=NW2_ERS12_msnw2ers START_PROFILE="/sapmnt/NW2/profile/NW2_ERS12_msnw2ers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW2_AERS

    sudo pcs constraint colocation add g-NW2_AERS with g-NW2_ASCS -5000
    sudo pcs constraint location rsc_sap_NW2_ASCS10 rule score=2000 runs_ers_NW2 eq 1
    sudo pcs constraint order g-NW2_ASCS then g-NW2_AERS kind=Optional symmetrical=false

    sudo pcs resource create rsc_sap_NW3_ASCS20 SAPInstance \
    InstanceName=NW3_ASCS20_msnw3ascs START_PROFILE="/sapmnt/NW3/profile/NW3_ASCS20_msnw3ascs" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW3_ASCS

    sudo pcs resource create rsc_sap_NW3_ERS22 SAPInstance \
    InstanceName=NW3_ERS22_msnw3ers START_PROFILE="/sapmnt/NW3/profile/NW2_ERS22_msnw3ers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW3_AERS

    sudo pcs constraint colocation add g-NW3_AERS with g-NW3_ASCS -5000
    sudo pcs constraint location rsc_sap_NW3_ASCS20 rule score=2000 runs_ers_NW3 eq 1
    sudo pcs constraint order g-NW3_ASCS then g-NW3_AERS kind=Optional symmetrical=false

    sudo pcs property set maintenance-mode=false
    ```

   SAP introdujo una opción de compatibilidad con el servidor 2 de puesta en cola, incluida la replicación, a partir de la versión de SAP NW 7.52. A partir de la plataforma ABAP (versión 1809), el servidor 2 de puesta en cola está instalado de forma predeterminada. Consulte la nota de SAP [2630416](https://launchpad.support.sap.com/#/notes/2630416) para consultar la compatibilidad con el servidor 2 de puesta en cola.
   Si usa la arquitectura de servidor 2 de puesta en cola ([ENSA2](https://help.sap.com/viewer/cff8531bc1d9416d91bb6781e628d4e0/1709%20001/en-US/6d655c383abf4c129b0e5c8683e7ecd8.html)), defina los recursos para los sistemas SAP **NW2** y **NW3** tal como se indica:

    ```
     sudo pcs property set maintenance-mode=true

    sudo pcs resource create rsc_sap_NW2_ASCS10 SAPInstance \
    InstanceName=NW2_ASCS10_msnw2ascs START_PROFILE="/sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW2_ASCS

    sudo pcs resource create rsc_sap_NW2_ERS12 SAPInstance \
    InstanceName=NW2_ERS12_msnw2ers START_PROFILE="/sapmnt/NW2/profile/NW2_ERS12_msnw2ers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW2_AERS

    sudo pcs constraint colocation add g-NW2_AERS with g-NW2_ASCS -5000
    sudo pcs constraint order g-NW2_ASCS then g-NW2_AERS kind=Optional symmetrical=false
    sudo pcs constraint order start g-NW2_ASCS then stop g-NW2_AERS symmetrical=false

    sudo pcs resource create rsc_sap_NW3_ASCS20 SAPInstance \
    InstanceName=NW3_ASCS20_msnw3ascs START_PROFILE="/sapmnt/NW3/profile/NW3_ASCS20_msnw3ascs" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW3_ASCS

    sudo pcs resource create rsc_sap_NW3_ERS22 SAPInstance \
    InstanceName=NW3_ERS22_msnw3ers START_PROFILE="/sapmnt/NW3/profile/NW2_ERS22_msnw3ers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW3_AERS

    sudo pcs constraint colocation add g-NW3_AERS with g-NW3_ASCS -5000
    sudo pcs constraint order g-NW3_ASCS then g-NW3_AERS kind=Optional symmetrical=false
    sudo pcs constraint order start g-NW3_ASCS then stop g-NW3_AERS symmetrical=false

    sudo pcs property set maintenance-mode=false
    ```

   Si está actualizando desde una versión anterior y va a cambiar al servidor 2 de puesta en cola, consulte la nota de SAP [2641019](https://launchpad.support.sap.com/#/notes/2641019). 

   > [!NOTE]
   > Los tiempos de espera de la configuración anterior son solo ejemplos y puede ser necesario adaptarlos a la configuración específica de SAP. 

   Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.
   En el ejemplo siguiente se muestra el estado de los recursos de clúster después de haber agregado los sistemas SAP **NW2** y **NW3** al clúster. 

    ```
     sudo pcs status

    Online: [ rhelmsscl1 rhelmsscl2 ]

    Full list of resources:

    rsc_st_azure   (stonith:fence_azure_arm):      Started rhelmsscl1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    ```

8. **[A]** Agregue reglas de firewall para ASCS y ERS en ambos nodos.  En el ejemplo siguiente se muestran las reglas de firewall para los sistemas SAP **NW2** y **NW3**.  

   ```
    # NW2 - ASCS
    sudo firewall-cmd --zone=public --add-port=62010/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=62010/tcp
    sudo firewall-cmd --zone=public --add-port=3210/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3210/tcp
    sudo firewall-cmd --zone=public --add-port=3610/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3610/tcp
    sudo firewall-cmd --zone=public --add-port=3910/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3910/tcp
    sudo firewall-cmd --zone=public --add-port=8110/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=8110/tcp
    sudo firewall-cmd --zone=public --add-port=51013/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51013/tcp
    sudo firewall-cmd --zone=public --add-port=51014/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51014/tcp
    sudo firewall-cmd --zone=public --add-port=51016/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51016/tcp
    # NW2 - ERS
    sudo firewall-cmd --zone=public --add-port=62112/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=62112/tcp
    sudo firewall-cmd --zone=public --add-port=3312/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3312/tcp
    sudo firewall-cmd --zone=public --add-port=51213/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51213/tcp
    sudo firewall-cmd --zone=public --add-port=51214/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51214/tcp
    sudo firewall-cmd --zone=public --add-port=51216/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51216/tcp
    # NW3 - ASCS
    sudo firewall-cmd --zone=public --add-port=62020/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=62020/tcp
    sudo firewall-cmd --zone=public --add-port=3220/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3220/tcp
    sudo firewall-cmd --zone=public --add-port=3620/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3620/tcp
    sudo firewall-cmd --zone=public --add-port=3920/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3920/tcp
    sudo firewall-cmd --zone=public --add-port=8120/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=8120/tcp
    sudo firewall-cmd --zone=public --add-port=52013/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52013/tcp
    sudo firewall-cmd --zone=public --add-port=52014/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52014/tcp
    sudo firewall-cmd --zone=public --add-port=52016/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52016/tcp
    # NW3 - ERS
    sudo firewall-cmd --zone=public --add-port=62122/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=62122/tcp
    sudo firewall-cmd --zone=public --add-port=3322/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3322/tcp
    sudo firewall-cmd --zone=public --add-port=52213/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52213/tcp
    sudo firewall-cmd --zone=public --add-port=52214/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52214/tcp
    sudo firewall-cmd --zone=public --add-port=52216/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52216/tcp
   ```

### <a name="proceed-with-the-sap-installation"></a>Continuación de la instalación de SAP 

Complete la instalación de SAP mediante las siguientes tareas:

* [Preparación de los servidores de aplicaciones de SAP NetWeaver](./high-availability-guide-rhel-netapp-files.md#2d6008b0-685d-426c-b59e-6cd281fd45d7)
* [Instalación de una instancia de DBMS](./high-availability-guide-rhel-netapp-files.md#install-database)
* [Instalación de un servidor de aplicaciones de SAP principal](./high-availability-guide-rhel-netapp-files.md#sap-netweaver-application-server-installation)
* Instalación de una o varias instancias de aplicación de SAP adicionales

## <a name="test-the-multi-sid-cluster-setup"></a>Prueba de la configuración del clúster de varios SID

Las siguientes pruebas constituyen un subconjunto de los casos de prueba de las guías de procedimientos recomendados de Red Hat. Se incluyen en este artículo para su mayor comodidad. Para obtener una lista completa de las pruebas del clúster, consulte la siguiente documentación:

* Si se usan volúmenes NFS de Azure NetApp Files, siga [Alta disponibilidad de VM de Azure para SAP NetWeaver en RHEL con Azure NetApp Files para SAP Applications](./high-availability-guide-rhel-netapp-files.md).
* Si usa un clúster `GlusterFS` de alta disponibilidad, siga [Alta disponibilidad de VM de Azure para SAP NetWeaver en RHEL para SAP Applications](./high-availability-guide-rhel.md).  

Además, lea siempre las guías de procedimientos recomendados de Red Hat y realice todas las pruebas adicionales que puedan haberse agregado.  
Las pruebas que se presentan se encuentran en un clúster de dos nodos y varios SID con tres sistemas SAP instalados.  

1. Migración manual de la instancia de ASCS. En el ejemplo se muestra cómo migrar la instancia de ASCS para SAP System NW3.

   Estado del recurso antes de iniciar la prueba:

   ```
    Online: [ rhelmsscl1 rhelmsscl2 ]

    Full list of resources:

    rsc_st_azure   (stonith:fence_azure_arm):      Started rhelmsscl1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
   ```

   Ejecute los siguientes comandos como raíz para migrar la instancia de NW3 ASCS.

   ```
    pcs resource move rsc_sap_NW3_ASCS200
    # Clear temporary migration constraints
    pcs resource clear rsc_sap_NW3_ASCS20

    # Remove failed actions for the ERS that occurred as part of the migration
    pcs resource cleanup rsc_sap_NW3_ERS22
   ```

   Estado del recurso después de la prueba:

   ```
    Online: [ rhelmsscl1 rhelmsscl2 ]

    Full list of resources:

    rsc_st_azure   (stonith:fence_azure_arm):      Started rhelmsscl1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
   ```

1. Simulación de bloqueo de nodo

   Estado del recurso antes de iniciar la prueba:

   ```
    Online: [ rhelmsscl1 rhelmsscl2 ]

    Full list of resources:

    rsc_st_azure   (stonith:fence_azure_arm):      Started rhelmsscl1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
   ```

   Ejecute el siguiente comando como raíz en el nodo en el que haya al menos una instancia de ASCS en ejecución. En este ejemplo, se ejecutó el comando en `rhelmsscl1`, donde las instancias de ASCS para NW1,NW2 y NW3 están en ejecución.  

   ```
   echo c > /proc/sysrq-trigger
   ```

   El estado tras la prueba y tras el nodo, que se bloqueó e inició de nuevo, debería tener el siguiente aspecto.

   ```
    Full list of resources:

    rsc_st_azure    (stonith:fence_azure_arm):      Started rhelmsscl2
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
   ```

   Si hay mensajes para recursos con errores, limpie el estado de dichos recursos. Por ejemplo:

   ```
   pcs resource cleanup rsc_sap_NW1_ERS02
   ```

## <a name="next-steps"></a>Pasos siguientes

* [Planeamiento e implementación de Azure Virtual Machines para SAP][planning-guide]
* [Implementación de Azure Virtual Machines para SAP][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP][dbms-guide]
* Para más información sobre cómo establecer la alta disponibilidad y planear la recuperación ante desastres de SAP HANA en Azure Virtual Machines, consulte [Alta disponibilidad de SAP HANA en las máquinas virtuales de Azure][sap-hana-ha].
