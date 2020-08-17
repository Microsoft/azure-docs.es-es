---
title: GlusterFS en máquinas virtuales de Azure en RHEL para SAP NetWeaver | Microsoft Docs
description: GlusterFS en máquinas virtuales de Azure de Red Hat Enterprise Linux para SAP NetWeaver
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
ms.date: 08/16/2018
ms.author: radeltch
ms.openlocfilehash: e6b8708edeeb4b3b69dab82265d13db2a4c22e08
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87051656"
---
# <a name="glusterfs-on-azure-vms-on-red-hat-enterprise-linux-for-sap-netweaver"></a>GlusterFS en máquinas virtuales de Azure de Red Hat Enterprise Linux para SAP NetWeaver

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2009879]:https://launchpad.support.sap.com/#/notes/2009879
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1999351]:https://launchpad.support.sap.com/#/notes/1999351

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[template-file-server]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-file-server-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability-rhel.md

En este artículo se describe cómo implementar y configurar las máquinas virtuales y cómo instalar el clúster de GlusterFS que se puede usar para almacenar los datos compartidos de un sistema SAP con alta disponibilidad.
En esta guía se describe cómo configurar un clúster de GlusterFS que se usa en dos sistemas SAP: NW1 y NW2. En los nombres de los recursos (por ejemplo, máquinas virtuales o redes virtuales) del ejemplo se da por supuesto que ha usado la [plantilla del servidor de archivos de SAP][template-file-server] con el prefijo de recursos **glust**.

Lea primero las notas y los documentos de SAP siguientes:

* Nota de SAP [1928533], que incluye:
  * La lista de tamaños de máquina virtual de Azure que se admiten para la implementación del software de SAP
  * Información importante sobre la capacidad para los tamaños de máquina virtual de Azure
  * Software de SAP admitido y combinaciones de sistema operativo y base de datos
  * Versión del kernel de SAP requerida para Windows y Linux en Microsoft Azure

* La nota de SAP [2015553] enumera los requisitos previos para las implementaciones de software de SAP admitidas por SAP en Azure.
* La nota de SAP [2002167] recomienda la configuración del sistema operativo para Red Hat Enterprise Linux
* La nota de SAP [2009879] contiene las instrucciones de SAP HANA para Red Hat Enterprise Linux
* La nota de SAP [2178632] contiene información detallada sobre todas las métricas de supervisión notificadas para SAP en Azure.
* La nota de SAP [2191498] incluye la versión de SAP Host Agent necesaria para Linux en Azure.
* La nota de SAP [2243692] incluye información acerca de las licencias de SAP en Linux en Azure.
* La nota de SAP [1999351] contiene más soluciones de problemas de la extensión de supervisión mejorada de Azure para SAP.
* La [WIKI de la comunidad SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contiene todas las notas de SAP que se necesitan para Linux.
* [Planeación e implementación de Azure Virtual Machines para SAP en Linux][planning-guide]
* [Implementación de Azure Virtual Machines para SAP en Linux (este artículo)][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP en Linux][dbms-guide]
* [Documentación de producto para Red Hat Gluster Storage](https://access.redhat.com/documentation/red_hat_gluster_storage/)
* Documentación general de RHEL
  * [Introducción al complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [Administración del complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [Referencia del complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
* Documentación de RHEL específica para Azure:
  * [Directivas de compatibilidad para clústeres de alta disponibilidad RHEL: instancias de Microsoft Azure Virtual Machines como miembros del clúster](https://access.redhat.com/articles/3131341)
  * [Instalación y configuración de un clúster de alta disponibilidad de Red Hat Enterprise Linux 7.4 (y versiones posteriores) en Microsoft Azure](https://access.redhat.com/articles/3252491)

## <a name="overview"></a>Información general

Para lograr alta disponibilidad, SAP NetWeaver requiere almacenamiento compartido. GlusterFS está configurado en un clúster distinto y lo pueden usar varios sistemas SAP.

![Información general sobre la alta disponibilidad de SAP NetWeaver](./media/high-availability-guide-rhel-glusterfs/rhel-glusterfs.png)

## <a name="set-up-glusterfs"></a>Configuración de GlusterFS

Puede usar una plantilla de Azure de GitHub para implementar todos los recursos de Azure necesarios, como máquinas virtuales, conjuntos de disponibilidad e interfaces de red, o puede implementar los recursos manualmente.

### <a name="deploy-linux-via-azure-template"></a>Implementación de Linux mediante una plantilla de Azure

Azure Marketplace contiene una imagen de Red Hat Enterprise Linux que puede usar para implementar nuevas máquinas virtuales.
Para implementar todos los recursos necesarios, puede usar una de las plantillas de inicio rápido de GitHub. La plantilla implementa las máquinas virtuales y el conjunto de disponibilidad, entre otros. Siga estos pasos para implementar la plantilla:

1. Abra la [plantilla del servidor de archivos SAP][template-file-server] en Azure Portal
1. Escriba los siguientes parámetros:
   1. Prefijo de recurso  
      Escriba el prefijo que desea usar. El valor se usa como prefijo de los recursos que se implementan.
   2. Número de sistemas SAP: escriba el número de sistemas SAP que usarán este servidor de archivos. Esta acción implementará el número necesario de discos, entre otras cosas.
   3. Tipo de sistema operativo  
      Seleccione una de las distribuciones de Linux. En este ejemplo, seleccione RHEL 7.
   4. Nombre de usuario administrador, contraseña del administrador o clave SSH  
      Se crea un usuario nuevo que se puede usar para iniciar sesión en la máquina.
   5. Identificador de subred  
      Si quiere implementar la máquina virtual en una red virtual existente en la que tiene una subred definida a la que se debe asignar la máquina virtual, asigne un nombre al identificador de esa subred específica. El identificador suele tener este aspecto: /subscriptions/ **&lt;id. de suscripción&gt;** /resourceGroups/ **&lt;nombre del grupo de recursos&gt;** /providers/Microsoft.Network/virtualNetworks/ **&lt;nombre de red virtual&gt;** /subnets/ **&lt;nombre de subred&gt;**

### <a name="deploy-linux-manually-via-azure-portal"></a>Implementación manual de Linux mediante Azure Portal

En primer lugar, debe crear las máquinas virtuales de este clúster. Después, creará un equilibrador de carga y usará las máquinas virtuales de los grupos de servidores back-end. Se recomienda que sea un [equilibrador de carga estándar](../../../load-balancer/load-balancer-overview.md).  

1. Creación de un grupo de recursos
1. Creación de una red virtual
1. Creación de un conjunto de disponibilidad  
   Establecimiento del dominio máximo de actualización
1. Creación de la máquina virtual 1  
   Use al menos RHEL 7; en este ejemplo se usó la imagen de Red Hat Enterprise Linux 7.4. <https://portal.azure.com/#create/RedHat.RedHatEnterpriseLinux74-ARM>  
   Seleccione el conjunto de disponibilidad creado anteriormente.  
1. Creación de la máquina virtual 2  
   Use al menos RHEL 7; en este ejemplo se usó la imagen de Red Hat Enterprise Linux 7.4. <https://portal.azure.com/#create/RedHat.RedHatEnterpriseLinux74-ARM>  
   Seleccione el conjunto de disponibilidad creado anteriormente.  
1. Agregue un disco de datos por cada sistema SAP a ambas máquinas virtuales.

### <a name="configure-glusterfs"></a>Configuración de GlusterFS

Los elementos siguientes tienen el prefijo **[A]** : aplicable a todos los nodos; **[1]** : aplicable solo al nodo 1; **[2]** : aplicable solo al nodo 2 o **[3]** : aplicable solo al nodo 3.

1. **[A]** Configure la resolución nombres de host

   Puede usar un servidor DNS o modificar /etc/hosts en todos los nodos. En este ejemplo se muestra cómo utilizar el archivo /etc/hosts.
   Reemplace la dirección IP y el nombre de host en los siguientes comandos

   <pre><code>sudo vi /etc/hosts
   </code></pre>

   Inserte las siguientes líneas en /etc/hosts. Cambie la dirección IP y el nombre de host para que coincida con su entorno

   <pre><code># IP addresses of the Gluster nodes
   <b>10.0.0.40 glust-0</b>
   <b>10.0.0.41 glust-1</b>
   <b>10.0.0.42 glust-2</b>
   </code></pre>

1. **[A]** Registro

   Registre las máquinas virtuales y asócielas a un grupo que contenga repositorios para RHEL 7 y GlusterFS

   <pre><code>sudo subscription-manager register
   sudo subscription-manager attach --pool=&lt;pool id&gt;
   </code></pre>

1. **[A]** Habilitación de los repositorios de GlusterFS

   Para instalar los paquetes necesarios, habilite los siguientes repositorios.

   <pre><code>sudo subscription-manager repos --disable "*"
   sudo subscription-manager repos --enable=rhel-7-server-rpms
   sudo subscription-manager repos --enable=rh-gluster-3-for-rhel-7-server-rpms
   </code></pre>
  
1. **[A]** Instalación de los paquetes de GlusterFS

   Instale estos paquetes en todos los nodos de GlusterFS

   <pre><code>sudo yum -y install redhat-storage-server
   </code></pre>

   Reinicie los nodos después de la instalación.

1. **[A]** Modificación de las reglas de firewall

   Agregue reglas de firewall para permitir el tráfico de cliente a los nodos de GlusterFS.

   <pre><code># list the available zones
   firewall-cmd --get-active-zones
   
   sudo firewall-cmd --zone=public --add-service=glusterfs --permanent
   sudo firewall-cmd --zone=public --add-service=glusterfs
   </code></pre>

1. **[A]** Habilitación e inicio del servicio de GlusterFS

   Inicie el servicio de GlusterFS en todos los nodos.

   <pre><code>sudo systemctl start glusterd
   sudo systemctl enable glusterd
   </code></pre>

1. **[1]** Creación de un GlusterFS

   Ejecute los siguientes comandos para crear el clúster de GlusterFS

   <pre><code>sudo gluster peer probe glust-1
   sudo gluster peer probe glust-2
   
   # Check gluster peer status
   sudo gluster peer status
   
   # Number of Peers: 2
   # 
   # Hostname: glust-1
   # Uuid: 10d43840-fee4-4120-bf5a-de9c393964cd
   # State: Accepted peer request (Connected)
   # 
   # Hostname: glust-2
   # Uuid: 9e340385-12fe-495e-ab0f-4f851b588cba
   # State: Accepted peer request (Connected)
   </code></pre>

1. **[2]** Prueba del estado del mismo nivel

   Compruebe el estado del mismo nivel en el segundo nodo

   <pre><code>sudo gluster peer status
   # Number of Peers: 2
   #
   # Hostname: glust-0
   # Uuid: 6bc6927b-7ee2-461b-ad04-da123124d6bd
   # State: Peer in Cluster (Connected)
   #
   # Hostname: glust-2
   # Uuid: 9e340385-12fe-495e-ab0f-4f851b588cba
   # State: Peer in Cluster (Connected)
   </code></pre>

1. **[3]** Prueba del estado del mismo nivel

   Compruebe el estado del mismo nivel en el tercer nodo

   <pre><code>sudo gluster peer status
   # Number of Peers: 2
   #
   # Hostname: glust-0
   # Uuid: 6bc6927b-7ee2-461b-ad04-da123124d6bd
   # State: Peer in Cluster (Connected)
   #
   # Hostname: glust-1
   # Uuid: 10d43840-fee4-4120-bf5a-de9c393964cd
   # State: Peer in Cluster (Connected)
   </code></pre>

1. **[A]** Creación de LVM

   En este ejemplo, el GlusterFS se usa en dos sistemas SAP: NW1 y NW2. Use los siguientes comandos para crear configuraciones de LVM para estos sistemas SAP.

   Use estos comandos para el sistema NW1

   <pre><code>sudo pvcreate --dataalignment 1024K /dev/disk/azure/scsi1/lun0
   sudo pvscan
   sudo vgcreate --physicalextentsize 256K rhgs-<b>NW1</b> /dev/disk/azure/scsi1/lun0
   sudo vgscan
   sudo lvcreate -l 50%FREE -n rhgs-<b>NW1</b>/sapmnt
   sudo lvcreate -l 20%FREE -n rhgs-<b>NW1</b>/trans
   sudo lvcreate -l 10%FREE -n rhgs-<b>NW1</b>/sys
   sudo lvcreate -l 50%FREE -n rhgs-<b>NW1</b>/ascs
   sudo lvcreate -l 100%FREE -n rhgs-<b>NW1</b>/aers
   sudo lvscan
   
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW1</b>/sapmnt
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW1</b>/trans
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW1</b>/sys
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW1</b>/ascs
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW1</b>/aers
   
   sudo mkdir -p /rhs/<b>NW1</b>/sapmnt
   sudo mkdir -p /rhs/<b>NW1</b>/trans
   sudo mkdir -p /rhs/<b>NW1</b>/sys
   sudo mkdir -p /rhs/<b>NW1</b>/ascs
   sudo mkdir -p /rhs/<b>NW1</b>/aers
   
   sudo chattr +i /rhs/<b>NW1</b>/sapmnt
   sudo chattr +i /rhs/<b>NW1</b>/trans
   sudo chattr +i /rhs/<b>NW1</b>/sys
   sudo chattr +i /rhs/<b>NW1</b>/ascs
   sudo chattr +i /rhs/<b>NW1</b>/aers

   echo -e "/dev/rhgs-<b>NW1</b>/sapmnt\t/rhs/<b>NW1</b>/sapmnt\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW1</b>/trans\t/rhs/<b>NW1</b>/trans\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW1</b>/sys\t/rhs/<b>NW1</b>/sys\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW1</b>/ascs\t/rhs/<b>NW1</b>/ascs\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW1</b>/aers\t/rhs/<b>NW1</b>/aers\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   sudo mount -a
   </code></pre>

   Use estos comandos para el sistema NW2

   <pre><code>sudo pvcreate --dataalignment 1024K /dev/disk/azure/scsi1/lun1
   sudo pvscan
   sudo vgcreate --physicalextentsize 256K rhgs-<b>NW2</b> /dev/disk/azure/scsi1/lun1
   sudo vgscan
   sudo lvcreate -l 50%FREE -n rhgs-<b>NW2</b>/sapmnt
   sudo lvcreate -l 20%FREE -n rhgs-<b>NW2</b>/trans
   sudo lvcreate -l 10%FREE -n rhgs-<b>NW2</b>/sys
   sudo lvcreate -l 50%FREE -n rhgs-<b>NW2</b>/ascs
   sudo lvcreate -l 100%FREE -n rhgs-<b>NW2</b>/aers
   
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW2</b>/sapmnt
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW2</b>/trans
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW2</b>/sys
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW2</b>/ascs
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW2</b>/aers
   
   sudo mkdir -p /rhs/<b>NW2</b>/sapmnt
   sudo mkdir -p /rhs/<b>NW2</b>/trans
   sudo mkdir -p /rhs/<b>NW2</b>/sys
   sudo mkdir -p /rhs/<b>NW2</b>/ascs
   sudo mkdir -p /rhs/<b>NW2</b>/aers
   
   sudo chattr +i /rhs/<b>NW2</b>/sapmnt
   sudo chattr +i /rhs/<b>NW2</b>/trans
   sudo chattr +i /rhs/<b>NW2</b>/sys
   sudo chattr +i /rhs/<b>NW2</b>/ascs
   sudo chattr +i /rhs/<b>NW2</b>/aers
   sudo lvscan
   
   echo -e "/dev/rhgs-<b>NW2</b>/sapmnt\t/rhs/<b>NW2</b>/sapmnt\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW2</b>/trans\t/rhs/<b>NW2</b>/trans\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW2</b>/sys\t/rhs/<b>NW2</b>/sys\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW2</b>/ascs\t/rhs/<b>NW2</b>/ascs\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW2</b>/aers\t/rhs/<b>NW2</b>/aers\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   sudo mount -a
   </code></pre>

1. **[1]** Creación del volumen distribuido

   Use los siguientes comandos para crear el volumen de GlusterFS para el sistema NW1 e inícielo.

   <pre><code>sudo gluster vol create <b>NW1</b>-sapmnt replica 3 glust-0:/rhs/<b>NW1</b>/sapmnt glust-1:/rhs/<b>NW1</b>/sapmnt glust-2:/rhs/<b>NW1</b>/sapmnt force
   sudo gluster vol create <b>NW1</b>-trans replica 3 glust-0:/rhs/<b>NW1</b>/trans glust-1:/rhs/<b>NW1</b>/trans glust-2:/rhs/<b>NW1</b>/trans force
   sudo gluster vol create <b>NW1</b>-sys replica 3 glust-0:/rhs/<b>NW1</b>/sys glust-1:/rhs/<b>NW1</b>/sys glust-2:/rhs/<b>NW1</b>/sys force
   sudo gluster vol create <b>NW1</b>-ascs replica 3 glust-0:/rhs/<b>NW1</b>/ascs glust-1:/rhs/<b>NW1</b>/ascs glust-2:/rhs/<b>NW1</b>/ascs force
   sudo gluster vol create <b>NW1</b>-aers replica 3 glust-0:/rhs/<b>NW1</b>/aers glust-1:/rhs/<b>NW1</b>/aers glust-2:/rhs/<b>NW1</b>/aers force
   
   sudo gluster volume start <b>NW1</b>-sapmnt
   sudo gluster volume start <b>NW1</b>-trans
   sudo gluster volume start <b>NW1</b>-sys
   sudo gluster volume start <b>NW1</b>-ascs
   sudo gluster volume start <b>NW1</b>-aers
   </code></pre>

   Use los siguientes comandos para crear el volumen de GlusterFS para el sistema NW2 e inícielo.

   <pre><code>sudo gluster vol create <b>NW2</b>-sapmnt replica 3 glust-0:/rhs/<b>NW2</b>/sapmnt glust-1:/rhs/<b>NW2</b>/sapmnt glust-2:/rhs/<b>NW2</b>/sapmnt force
   sudo gluster vol create <b>NW2</b>-trans replica 3 glust-0:/rhs/<b>NW2</b>/trans glust-1:/rhs/<b>NW2</b>/trans glust-2:/rhs/<b>NW2</b>/trans force
   sudo gluster vol create <b>NW2</b>-sys replica 3 glust-0:/rhs/<b>NW2</b>/sys glust-1:/rhs/<b>NW2</b>/sys glust-2:/rhs/<b>NW2</b>/sys force
   sudo gluster vol create <b>NW2</b>-ascs replica 3 glust-0:/rhs/<b>NW2</b>/ascs glust-1:/rhs/<b>NW2</b>/ascs glust-2:/rhs/<b>NW2</b>/ascs force
   sudo gluster vol create <b>NW2</b>-aers replica 3 glust-0:/rhs/<b>NW2</b>/aers glust-1:/rhs/<b>NW2</b>/aers glust-2:/rhs/<b>NW2</b>/aers force
   
   sudo gluster volume start <b>NW2</b>-sapmnt
   sudo gluster volume start <b>NW2</b>-trans
   sudo gluster volume start <b>NW2</b>-sys
   sudo gluster volume start <b>NW2</b>-ascs
   sudo gluster volume start <b>NW2</b>-aers
   </code></pre>

## <a name="next-steps"></a>Pasos siguientes

* [Instalación de SAP ASCS y la base de datos](high-availability-guide-rhel.md)
* [Planeamiento e implementación de Azure Virtual Machines para SAP][planning-guide]
* [Implementación de Azure Virtual Machines para SAP][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP][dbms-guide]
* Para obtener información sobre cómo establecer la alta disponibilidad y planear la recuperación ante desastres de SAP HANA en Azure (instancias grandes), vea [Alta disponibilidad y recuperación ante desastres de SAP HANA en Azure (instancias grandes)](hana-overview-high-availability-disaster-recovery.md).
* Para más información sobre cómo establecer la alta disponibilidad y planear la recuperación ante desastres de SAP HANA en Azure Virtual Machines, consulte [Alta disponibilidad de SAP HANA en las máquinas virtuales de Azure][sap-hana-ha].
