---
title: Alta disponibilidad para NFS en máquinas virtuales de Azure en SLES | Microsoft Docs
description: Alta disponibilidad para NFS en máquinas virtuales de Azure en SUSE Linux Enterprise Server
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
ms.date: 03/26/2020
ms.author: radeltch
ms.openlocfilehash: d522d66642abf55e478cea7579e36bdc64a8cf79
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87085170"
---
# <a name="high-availability-for-nfs-on-azure-vms-on-suse-linux-enterprise-server"></a>Alta disponibilidad para NFS en máquinas virtuales de Azure en SUSE Linux Enterprise Server

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

[sles-hae-guides]:https://www.suse.com/documentation/sle-ha-12/
[sles-for-sap-bp]:https://www.suse.com/documentation/sles-for-sap-12/
[suse-ha-12sp3-relnotes]:https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP3/

[template-multisid-xscs]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[template-converged]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-converged-md%2Fazuredeploy.json
[template-file-server]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-file-server-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability.md

En este artículo se describe cómo implementar y configurar las máquinas virtuales, instalar el marco de trabajo del clúster e instalar un servidor NFS de alta disponibilidad que se puede usar para almacenar los datos compartidos de un sistema SAP con alta disponibilidad.
En esta guía se describe cómo configurar un servidor NFS de alta disponibilidad que se usa en dos sistemas SAP: NW1 y NW2. En los nombres de los recursos (por ejemplo, máquinas virtuales, redes virtuales) del ejemplo se da por supuesto que ha usado la [plantilla del servidor de archivos de SAP][template-file-server] con el prefijo de recursos **prod**.

Lea primero las notas y los documentos de SAP siguientes:

* Nota de SAP [1928533], que incluye:
  * La lista de tamaños de máquina virtual de Azure que se admiten para la implementación del software de SAP
  * Información importante sobre la capacidad para los tamaños de máquina virtual de Azure
  * Software de SAP admitido y combinaciones de sistema operativo y base de datos
  * Versión del kernel de SAP requerida para Windows y Linux en Microsoft Azure

* La nota de SAP [2015553] enumera los requisitos previos para las implementaciones de software de SAP admitidas por SAP en Azure.
* La nota de SAP [2205917] contiene configuraciones recomendadas del sistema operativo para SUSE Linux Enterprise Server para SAP Applications
* La nota de SAP [1944799] contiene guías de SAP HANA para SUSE Linux Enterprise Server para SAP Applications
* La nota de SAP [2178632] contiene información detallada sobre todas las métricas de supervisión notificadas para SAP en Azure.
* La nota de SAP [2191498] incluye la versión de SAP Host Agent necesaria para Linux en Azure.
* La nota de SAP [2243692] incluye información acerca de las licencias de SAP en Linux en Azure.
* La nota de SAP [1984787] incluye información general sobre SUSE Linux Enterprise Server 12.
* La nota de SAP [1999351] contiene más soluciones de problemas de la extensión de supervisión mejorada de Azure para SAP.
* La [WIKI de la comunidad SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contiene todas las notas de SAP que se necesitan para Linux.
* [Planeación e implementación de Azure Virtual Machines para SAP en Linux][planning-guide]
* [Implementación de Azure Virtual Machines para SAP en Linux (este artículo)][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP en Linux][dbms-guide]
* [Guía de prácticas recomendadas para la extensión de alta disponibilidad 12 SP3 para SUSE Linux Enterprise][sles-hae-guides]
  * Almacenamiento NFS de alta disponibilidad con DRBD y Pacemaker
* [Guías de procedimientos recomendados de SUSE Linux Enterprise Server for SAP Applications 12 SP3][sles-for-sap-bp]
* [Notas de la versión de la Extensión 12 SP3 de alta disponibilidad para SUSE][suse-ha-12sp3-relnotes]

## <a name="overview"></a>Información general

Para lograr alta disponibilidad, SAP NetWeaver requiere un servidor NFS. El servidor NFS está configurado en un clúster distinto y lo pueden usar varios sistemas SAP.

![Información general sobre la alta disponibilidad de SAP NetWeaver](./media/high-availability-guide-nfs/ha-suse-nfs.png)

El servidor NFS usa un nombre de host virtual dedicado y direcciones IP virtuales para todos los sistemas SAP que usan este servidor NFS. En Azure, se requiere un equilibrador de carga para usar una dirección IP virtual. En la lista siguiente se muestra la configuración del equilibrador de carga.        

* Configuración de front-end
  * Dirección IP 10.0.0.4 de NW1
  * Dirección IP 10.0.0.5 de NW2
* Configuración de back-end
  * Se conecta a interfaces de red principales de todas las máquinas que deben ser parte del clúster NFS
* Puerto de sondeo
  * Puerto 61000 de NW1
  * Puerto 61001 de NW2
* Reglas de equilibrio de carga (si se usa un equilibrador de carga básico)
  * TCP 2049 de NW1
  * UDP 2049 de NW1
  * TCP 2049 de NW2
  * UDP 2049 de NW2

## <a name="set-up-a-highly-available-nfs-server"></a>Configuración de un servidor NFS de alta disponibilidad

Puede usar una plantilla de Azure de GitHub para implementar todos los recursos de Azure necesarios, incluidas las máquinas virtuales, el conjunto de disponibilidad y el equilibrador de carga, o puede implementar los recursos manualmente.

### <a name="deploy-linux-via-azure-template"></a>Implementación de Linux mediante una plantilla de Azure

Azure Marketplace contiene una imagen de SUSE Linux Enterprise Server para SAP Applications 12 que puede usar para implementar nuevas máquinas virtuales.
Para implementar todos los recursos necesarios, puede usar una de las plantillas de inicio rápido de GitHub. La plantilla implementa las máquinas virtuales, el equilibrador de carga, el conjunto de disponibilidad, etc. Siga estos pasos para implementar la plantilla:

1. Abra la [plantilla del servidor de archivos SAP][template-file-server] en Azure Portal   
1. Escriba los siguientes parámetros:
   1. Prefijo de recurso  
      Escriba el prefijo que desea usar. El valor se usa como prefijo de los recursos que se implementan.
   2. Recuento de sistema SAP  
      Escriba el número de sistemas SAP que este servidor de archivos va a utilizar. Esta acción implementará la cantidad necesaria de configuraciones de front-end, reglas de equilibrio de carga, puertos de sondeo, discos, etc.
   3. Tipo de sistema operativo  
      Seleccione una de las distribuciones de Linux. En este ejemplo, seleccione SLES 12
   4. Nombre de usuario y contraseña del administrador  
      Se crea un usuario nuevo que se puede usar para iniciar sesión en la máquina.
   5. Identificador de subred  
      Si quiere implementar la máquina virtual en una red virtual existente en la que tiene una subred definida a la que se debe asignar la máquina virtual, asigne un nombre al identificador de esa subred específica. El identificador suele tener este aspecto: /subscriptions/ **&lt;id. de suscripción&gt;** /resourceGroups/ **&lt;nombre del grupo de recursos&gt;** /providers/Microsoft.Network/virtualNetworks/ **&lt;nombre de red virtual&gt;** /subnets/ **&lt;nombre de subred&gt;**

### <a name="deploy-linux-manually-via-azure-portal"></a>Implementación manual de Linux mediante Azure Portal

En primer lugar, debe crear las máquinas virtuales de este clúster NFS. Después, creará un equilibrador de carga y usará las máquinas virtuales de los grupos de servidores back-end.

1. Creación de un grupo de recursos
1. Creación de una red virtual
1. Creación de un conjunto de disponibilidad  
   Establecimiento del dominio máximo de actualización
1. Creación de la máquina virtual 1 Use al menos SLES4SAP 12 SP3, en este ejemplo se usa la imagen SLES4SAP 12 SP3 BYOS SLES para SAP Applications 12 SP3 (BYOS)  
   Seleccione el conjunto de disponibilidad creado anteriormente.  
1. Creación de la máquina virtual 2 Use al menos SLES4SAP 12 SP3, en este ejemplo se usa la imagen SLES4SAP 12 SP3 BYOS  
   Se usa SLES para SAP Applications 12 SP3 (BYOS)  
   Seleccione el conjunto de disponibilidad creado anteriormente.  
1. Agregue un disco de datos por cada sistema SAP a ambas máquinas virtuales.
1. Cree un equilibrador de carga (interno). Se recomienda que sea un [equilibrador de carga estándar](../../../load-balancer/load-balancer-overview.md).  
   1. Siga estas instrucciones para crear un equilibrador de carga estándar:
      1. Creación de las direcciones IP de front-end
         1. Dirección IP 10.0.0.4 de NW1
            1. Abra el equilibrador de carga, seleccione el grupo de direcciones IP de front-end y haga clic en Agregar
            1. Escriba el nombre del nuevo grupo IP de direcciones front-end (por ejemplo, **nw1-frontend**).
            1. Configure la asignación como estática y escriba la dirección IP (por ejemplo **10.0.0.4**).
            1. Haga clic en Aceptar
         1. Dirección IP 10.0.0.5 de NW2
            * Repita los pasos anteriores con NW2.
      1. Creación de los grupos de servidores back-end
         1. Se conecta a interfaces de red principales de todas las máquinas que deben ser parte del clúster NFS
            1. Abra el equilibrador de carga, seleccione los grupos de back-end y haga clic en Agregar
            1. Escriba el nombre del nuevo grupo de servidores back-end (por ejemplo, **nw-backend**).
            1. Seleccione Virtual Network.
            1. Haga clic en Agregar una máquina virtual
            1. Seleccione las máquinas virtuales del clúster NFS y sus direcciones IP.
            1. Haga clic en Agregar.
      1. Creación de los sondeos de estado
         1. Puerto 61000 de NW1
            1. Abra el equilibrador de carga, seleccione los sondeos de estado y haga clic en Agregar
            1. Escriba el nombre del nuevo sondeo de estado (por ejemplo, **nw1-hp**).
            1. Seleccione TCP como protocolo, puerto 610**00**, y mantenga el intervalo de 5 y el umbral incorrecto 2.
            1. Haga clic en Aceptar
         1. Puerto 61001 de NW2
            * Repita los pasos anteriores para crear un sondeo de estado para NW2.
      1. Reglas de equilibrio de carga
         1. Abra el equilibrador de carga, seleccione las reglas de equilibrio de carga y haga clic en Agregar.
         1. Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, **nw1-lb**).
         1. Seleccione la dirección IP de front-end, el grupo de servidores back-end y el sondeo de estado que creó anteriormente (por ejemplo, **nw1-front-end**, **nw-backend** y **nw1-hp**)
         1. Seleccione **Puertos HA**.
         1. Aumente el tiempo de espera de inactividad a 30 minutos
         1. **Asegúrese de habilitar la dirección IP flotante**
         1. Haga clic en Aceptar
         * Repita los pasos anteriores para crear una regla de equilibrio de carga para NW2.
   1. Como alternativa, si el escenario requiere un equilibrador de carga básico, siga estas instrucciones:
      1. Creación de las direcciones IP de front-end
         1. Dirección IP 10.0.0.4 de NW1
            1. Abra el equilibrador de carga, seleccione el grupo de direcciones IP de front-end y haga clic en Agregar
            1. Escriba el nombre del nuevo grupo IP de direcciones front-end (por ejemplo, **nw1-frontend**).
            1. Configure la asignación como estática y escriba la dirección IP (por ejemplo **10.0.0.4**).
            1. Haga clic en Aceptar
         1. Dirección IP 10.0.0.5 de NW2
            * Repita los pasos anteriores con NW2.
      1. Creación de los grupos de servidores back-end
         1. Se conecta a interfaces de red principales de todas las máquinas que deben ser parte del clúster NFS
            1. Abra el equilibrador de carga, seleccione los grupos de back-end y haga clic en Agregar
            1. Escriba el nombre del nuevo grupo de servidores back-end (por ejemplo, **nw-backend**).
            1. Haga clic en Agregar una máquina virtual
            1. Seleccione el conjunto de disponibilidad que creó anteriormente
            1. Seleccione las máquinas virtuales del clúster NFS.
            1. Haga clic en Aceptar
      1. Creación de los sondeos de estado
         1. Puerto 61000 de NW1
            1. Abra el equilibrador de carga, seleccione los sondeos de estado y haga clic en Agregar
            1. Escriba el nombre del nuevo sondeo de estado (por ejemplo, **nw1-hp**).
            1. Seleccione TCP como protocolo, puerto 610**00**, y mantenga el intervalo de 5 y el umbral incorrecto 2.
            1. Haga clic en Aceptar
         1. Puerto 61001 de NW2
            * Repita los pasos anteriores para crear un sondeo de estado para NW2.
      1. Reglas de equilibrio de carga
         1. TCP 2049 de NW1
            1. Abra el equilibrador de carga, seleccione las reglas de equilibrio de carga y haga clic en Agregar
            1. Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, **nw1-lb-2049**).
            1. Seleccione la dirección IP de front-end, el grupo de servidores back-end y el sondeo de estado que creó anteriormente (por ejemplo, **nw1-front-end**).
            1. Conserve el protocolo **TCP** y escriba el puerto **2049**.
            1. Aumente el tiempo de espera de inactividad a 30 minutos
            1. **Asegúrese de habilitar la dirección IP flotante**
            1. Haga clic en Aceptar
         1. UDP 2049 de NW1
            * Repita los pasos anteriores con el puerto 2049 y UDP de NW1.
         1. TCP 2049 de NW2
            * Repita los pasos anteriores con el puerto 2049 y TCP de NW2.
         1. UDP 2049 de NW2
            * Repita los pasos anteriores con el puerto 2049 y UDP de NW2.

> [!Note]
> Cuando las máquinas virtuales sin direcciones IP públicas se colocan en el grupo de back-end de Standard Load Balancer interno (sin dirección IP pública), no hay conectividad saliente de Internet, a menos que se realice una configuración adicional para permitir el enrutamiento a puntos de conexión públicos. Para obtener más información sobre cómo obtener conectividad saliente, vea [Conectividad de punto de conexión público para máquinas virtuales con Azure Standard Load Balancer en escenarios de alta disponibilidad de SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

> [!IMPORTANT]
> No habilite las marcas de tiempo TCP en VM de Azure que se encuentren detrás de Azure Load Balancer. Si habilita las marcas de tiempo TCP provocará un error en los sondeos de estado. Establezca el parámetro **net.ipv4.tcp_timestamps** a **0**. Consulte [Sondeos de estado de Load Balancer](../../../load-balancer/load-balancer-custom-probe-overview.md) para obtener más información.

### <a name="create-pacemaker-cluster"></a>Creación del clúster de Pacemaker

Siga los pasos de [Configuración de Pacemaker en SUSE Linux Enterprise Server en Azure](high-availability-guide-suse-pacemaker.md) para crear un clúster de Pacemaker básico para este servidor NFS.

### <a name="configure-nfs-server"></a>Configuración del servidor NFS

Los elementos siguientes tienen el prefijo **[A]** : aplicable a todos los nodos, **[1]** : aplicable solo al nodo 1 o **[2]** : aplicable solo al nodo 2.

1. **[A]** Configure la resolución nombres de host

   Puede usar un servidor DNS o modificar /etc/hosts en todos los nodos. En este ejemplo se muestra cómo utilizar el archivo /etc/hosts.
   Reemplace la dirección IP y el nombre de host en los siguientes comandos

   <pre><code>sudo vi /etc/hosts
   </code></pre>
   
   Inserte las siguientes líneas en /etc/hosts. Cambie la dirección IP y el nombre de host para que coincida con su entorno
   
   <pre><code># IP address of the load balancer frontend configuration for NFS
   <b>10.0.0.4 nw1-nfs</b>
   <b>10.0.0.5 nw2-nfs</b>
   </code></pre>

1. **[A]** Habilite el servidor NFS.

   Cree la entrada de exportación de raíz NFS

   <pre><code>sudo sh -c 'echo /srv/nfs/ *\(rw,no_root_squash,fsid=0\)>/etc/exports'
   
   sudo mkdir /srv/nfs/
   </code></pre>

1. **[A]** Instale los componentes de drbd

   <pre><code>sudo zypper install drbd drbd-kmp-default drbd-utils
   </code></pre>

1. **[A]** Cree una partición para los dispositivos drbd.

   Enumere todos los discos de datos disponibles.

   <pre><code>sudo ls /dev/disk/azure/scsi1/
   </code></pre>

   Salida de ejemplo
   
   ```
   lun0  lun1
   ```

   Cree particiones para cada disco de datos.

   <pre><code>sudo sh -c 'echo -e "n\n\n\n\n\nw\n" | fdisk /dev/disk/azure/scsi1/lun0'
   sudo sh -c 'echo -e "n\n\n\n\n\nw\n" | fdisk /dev/disk/azure/scsi1/lun1'
   </code></pre>

1. **[A]** Cree configuraciones de LVM

   Enumere todas las particiones disponibles.

   <pre><code>ls /dev/disk/azure/scsi1/lun*-part*
   </code></pre>

   Salida de ejemplo
   
   ```
   /dev/disk/azure/scsi1/lun0-part1  /dev/disk/azure/scsi1/lun1-part1
   ```

   Cree volúmenes LVM para cada partición.

   <pre><code>sudo pvcreate /dev/disk/azure/scsi1/lun0-part1  
   sudo vgcreate vg-<b>NW1</b>-NFS /dev/disk/azure/scsi1/lun0-part1
   sudo lvcreate -l 100%FREE -n <b>NW1</b> vg-<b>NW1</b>-NFS

   sudo pvcreate /dev/disk/azure/scsi1/lun1-part1
   sudo vgcreate vg-<b>NW2</b>-NFS /dev/disk/azure/scsi1/lun1-part1
   sudo lvcreate -l 100%FREE -n <b>NW2</b> vg-<b>NW2</b>-NFS
   </code></pre>

1. **[A]** Configure drbd.

   <pre><code>sudo vi /etc/drbd.conf
   </code></pre>

   Asegúrese de que el archivo drbd.conf contenga las siguientes dos líneas:

   <pre><code>include "drbd.d/global_common.conf";
   include "drbd.d/*.res";
   </code></pre>

   Cambie la configuración de drbd global.

   <pre><code>sudo vi /etc/drbd.d/global_common.conf
   </code></pre>

   Agregue las siguientes entradas a la sección de red y controlador.

   <pre><code>global {
        usage-count no;
   }
   common {
        handlers {
             fence-peer "/usr/lib/drbd/crm-fence-peer.sh";
             after-resync-target "/usr/lib/drbd/crm-unfence-peer.sh";
             split-brain "/usr/lib/drbd/notify-split-brain.sh root";
             pri-lost-after-sb "/usr/lib/drbd/notify-pri-lost-after-sb.sh; /usr/lib/drbd/notify-emergency-reboot.sh; echo b > /proc/sysrq-trigger ; reboot -f";
        }
        startup {
             wfc-timeout 0;
        }
        options {
        }
        disk {
             md-flushes yes;
             disk-flushes yes;
             c-plan-ahead 1;
             c-min-rate 100M;
             c-fill-target 20M;
             c-max-rate 4G;
        }
        net {
             after-sb-0pri discard-younger-primary;
             after-sb-1pri discard-secondary;
             after-sb-2pri call-pri-lost-after-sb;
             protocol     C;
             tcp-cork yes;
             max-buffers 20000;
             max-epoch-size 20000;
             sndbuf-size 0;
             rcvbuf-size 0;
        }
   }
   </code></pre>

1. **[A]** Cree los dispositivos drbd de NFS.

   <pre><code>sudo vi /etc/drbd.d/<b>NW1</b>-nfs.res
   </code></pre>

   Inserte la configuración del nuevo dispositivo drbd y salga

   <pre><code>resource <b>NW1</b>-nfs {
        protocol     C;
        disk {
             on-io-error       detach;
        }
        on <b>prod-nfs-0</b> {
             address   <b>10.0.0.6:7790</b>;
             device    /dev/drbd<b>0</b>;
             disk      /dev/<b>vg-NW1-NFS</b>/<b>NW1</b>;
             meta-disk internal;
        }
        on <b>prod-nfs-1</b> {
             address   <b>10.0.0.7:7790</b>;
             device    /dev/drbd<b>0</b>;
             disk      /dev/<b>vg-NW1-NFS</b>/<b>NW1</b>;
             meta-disk internal;
        }
   }
   </code></pre>

   <pre><code>sudo vi /etc/drbd.d/<b>NW2</b>-nfs.res
   </code></pre>

   Inserte la configuración del nuevo dispositivo drbd y salga

   <pre><code>resource <b>NW2</b>-nfs {
        protocol     C;
        disk {
             on-io-error       detach;
        }
        on <b>prod-nfs-0</b> {
             address   <b>10.0.0.6:7791</b>;
             device    /dev/drbd<b>1</b>;
             disk      /dev/<b>vg-NW2-NFS</b>/<b>NW2</b>;
             meta-disk internal;
        }
        on <b>prod-nfs-1</b> {
             address   <b>10.0.0.7:7791</b>;
             device    /dev/drbd<b>1</b>;
             disk      /dev/<b>vg-NW2-NFS</b>/<b>NW2</b>;
             meta-disk internal;
        }
   }
   </code></pre>

   Cree el dispositivo drbd e inícielo

   <pre><code>sudo drbdadm create-md <b>NW1</b>-nfs
   sudo drbdadm create-md <b>NW2</b>-nfs
   sudo drbdadm up <b>NW1</b>-nfs
   sudo drbdadm up <b>NW2</b>-nfs
   </code></pre>

1. **[1]** Omita la sincronización inicial

   <pre><code>sudo drbdadm new-current-uuid --clear-bitmap <b>NW1</b>-nfs
   sudo drbdadm new-current-uuid --clear-bitmap <b>NW2</b>-nfs
   </code></pre>

1. **[1]** Establezca el nodo principal

   <pre><code>sudo drbdadm primary --force <b>NW1</b>-nfs
   sudo drbdadm primary --force <b>NW2</b>-nfs
   </code></pre>

1. **[1]** Espere hasta que se sincronicen los nuevos dispositivos drbd

   <pre><code>sudo drbdsetup wait-sync-resource NW1-nfs
   sudo drbdsetup wait-sync-resource NW2-nfs
   </code></pre>

1. **[1]** Cree sistemas de archivos en los dispositivos drbd

   <pre><code>sudo mkfs.xfs /dev/drbd0
   sudo mkdir /srv/nfs/NW1
   sudo chattr +i /srv/nfs/NW1
   sudo mount -t xfs /dev/drbd0 /srv/nfs/NW1
   sudo mkdir /srv/nfs/NW1/sidsys
   sudo mkdir /srv/nfs/NW1/sapmntsid
   sudo mkdir /srv/nfs/NW1/trans
   sudo mkdir /srv/nfs/NW1/ASCS
   sudo mkdir /srv/nfs/NW1/ASCSERS
   sudo mkdir /srv/nfs/NW1/SCS
   sudo mkdir /srv/nfs/NW1/SCSERS
   sudo umount /srv/nfs/NW1

   sudo mkfs.xfs /dev/drbd1
   sudo mkdir /srv/nfs/NW2
   sudo chattr +i /srv/nfs/NW2
   sudo mount -t xfs /dev/drbd1 /srv/nfs/NW2
   sudo mkdir /srv/nfs/NW2/sidsys
   sudo mkdir /srv/nfs/NW2/sapmntsid
   sudo mkdir /srv/nfs/NW2/trans
   sudo mkdir /srv/nfs/NW2/ASCS
   sudo mkdir /srv/nfs/NW2/ASCSERS
   sudo mkdir /srv/nfs/NW2/SCS
   sudo mkdir /srv/nfs/NW2/SCSERS
   sudo umount /srv/nfs/NW2
   </code></pre>

1. **[A]**  Configure la detección de cerebro dividido de drbd.

   Al usar drbd para sincronizar los datos de un host a otro, puede producirse lo que se conoce como cerebro dividido. Un escenario de cerebro dividido es aquel en el que ambos nodos del clúster promovieron el drbd a principal y perdieron la sincronización. Aunque es una situación poco frecuente, debe controlar y resolver un cerebro dividido lo antes posible. Por lo tanto, es importante recibir una notificación cuando suceda una situación así.

   Lea la [documentación oficial de drbd](https://www.linbit.com/drbd-user-guide/users-guide-drbd-8-4/#s-split-brain-notification) para saber cómo configurar una notificación de cerebro dividido.

   También es posible recuperarse automáticamente de un escenario de cerebro dividido. Para más información, lea las [directivas de recuperación automática de cerebro dividido](https://www.linbit.com/drbd-user-guide/users-guide-drbd-8-4/#s-automatic-split-brain-recovery-configuration).
   
### <a name="configure-cluster-framework"></a>Configuración de la plataforma del clúster

1. **[1]** Agregue los dispositivos drbd de NFS del sistema SAP NW1 a la configuración del clúster.

   > [!IMPORTANT]
   > Pruebas recientes han mostrado situaciones en las que netcat deja de responder a las solicitudes debido al trabajo pendiente y a su limitación para controlar solo una conexión. El recurso netcat deja de escuchar las solicitudes del equilibrador de carga de Azure y la dirección IP flotante deja de estar disponible.  
   > En el caso de los clústeres de Pacemaker existentes, en el pasado se recomendaba reemplazar netcat por socat. Actualmente se recomienda usar el agente de recursos azure-lb, que forma parte de los agentes de recursos de paquetes, con los siguientes requisitos de versión de paquete:
   > - En el caso de SLES 12 SP4/SP5, la versión debe ser, al menos, resource-agents-4.3.018.a7fb5035-3.30.1.  
   > - Para SLES 15/15 SP1, la versión debe ser al menos resource-agents-4.3.0184.6ee15eb2-4.13.1.  
   >
   > Tenga en cuenta que el cambio requerirá un breve tiempo de inactividad.  
   > En el caso de los clústeres de Pacemaker existentes, si la configuración ya se ha cambiado para usar socat, como se describe en [Protección de la detección del equilibrador de carga de Azure](https://www.suse.com/support/kb/doc/?id=7024128), no hay ningún requisito para cambiar inmediatamente al agente de recursos de azure-lb.

   <pre><code>sudo crm configure rsc_defaults resource-stickiness="200"

   # Enable maintenance mode
   sudo crm configure property maintenance-mode=true
   
   sudo crm configure primitive drbd_<b>NW1</b>_nfs \
     ocf:linbit:drbd \
     params drbd_resource="<b>NW1</b>-nfs" \
     op monitor interval="15" role="Master" \
     op monitor interval="30" role="Slave"
   
   sudo crm configure ms ms-drbd_<b>NW1</b>_nfs drbd_<b>NW1</b>_nfs \
     meta master-max="1" master-node-max="1" clone-max="2" \
     clone-node-max="1" notify="true" interleave="true"
   
   sudo crm configure primitive fs_<b>NW1</b>_sapmnt \
     ocf:heartbeat:Filesystem \
     params device=/dev/drbd0 \
     directory=/srv/nfs/<b>NW1</b>  \
     fstype=xfs \
     op monitor interval="10s"
   
   sudo crm configure primitive nfsserver systemd:nfs-server \
     op monitor interval="30s"
   sudo crm configure clone cl-nfsserver nfsserver

   sudo crm configure primitive exportfs_<b>NW1</b> \
     ocf:heartbeat:exportfs \
     params directory="/srv/nfs/<b>NW1</b>" \
     options="rw,no_root_squash,crossmnt" clientspec="*" fsid=1 wait_for_leasetime_on_stop=true op monitor interval="30s"
   
   sudo crm configure primitive vip_<b>NW1</b>_nfs \
     IPaddr2 \
     params ip=<b>10.0.0.4</b> cidr_netmask=<b>24</b> op monitor interval=10 timeout=20
   
   sudo crm configure primitive nc_<b>NW1</b>_nfs azure-lb port=<b>61000</b>
   
   sudo crm configure group g-<b>NW1</b>_nfs \
     fs_<b>NW1</b>_sapmnt exportfs_<b>NW1</b> nc_<b>NW1</b>_nfs vip_<b>NW1</b>_nfs
   
   sudo crm configure order o-<b>NW1</b>_drbd_before_nfs inf: \
     ms-drbd_<b>NW1</b>_nfs:promote g-<b>NW1</b>_nfs:start
   
   sudo crm configure colocation col-<b>NW1</b>_nfs_on_drbd inf: \
     g-<b>NW1</b>_nfs ms-drbd_<b>NW1</b>_nfs:Master
   </code></pre>

1. **[1]** Agregue los dispositivos drbd de NFS del sistema SAP NW2 a la configuración del clúster.

   <pre><code># Enable maintenance mode
   sudo crm configure property maintenance-mode=true
   
   sudo crm configure primitive drbd_<b>NW2</b>_nfs \
     ocf:linbit:drbd \
     params drbd_resource="<b>NW2</b>-nfs" \
     op monitor interval="15" role="Master" \
     op monitor interval="30" role="Slave"
   
   sudo crm configure ms ms-drbd_<b>NW2</b>_nfs drbd_<b>NW2</b>_nfs \
     meta master-max="1" master-node-max="1" clone-max="2" \
     clone-node-max="1" notify="true" interleave="true"
   
   sudo crm configure primitive fs_<b>NW2</b>_sapmnt \
     ocf:heartbeat:Filesystem \
     params device=/dev/drbd1 \
     directory=/srv/nfs/<b>NW2</b>  \
     fstype=xfs \
     op monitor interval="10s"
   
   sudo crm configure primitive exportfs_<b>NW2</b> \
     ocf:heartbeat:exportfs \
     params directory="/srv/nfs/<b>NW2</b>" \
     options="rw,no_root_squash,crossmnt" clientspec="*" fsid=2 wait_for_leasetime_on_stop=true op monitor interval="30s"
   
   sudo crm configure primitive vip_<b>NW2</b>_nfs \
     IPaddr2 \
     params ip=<b>10.0.0.5</b> cidr_netmask=<b>24</b> op monitor interval=10 timeout=20
   
   sudo crm configure primitive nc_<b>NW2</b>_nfs azure-lb port=<b>61001</b>
   
   sudo crm configure group g-<b>NW2</b>_nfs \
     fs_<b>NW2</b>_sapmnt exportfs_<b>NW2</b> nc_<b>NW2</b>_nfs vip_<b>NW2</b>_nfs
   
   sudo crm configure order o-<b>NW2</b>_drbd_before_nfs inf: \
     ms-drbd_<b>NW2</b>_nfs:promote g-<b>NW2</b>_nfs:start
   
   sudo crm configure colocation col-<b>NW2</b>_nfs_on_drbd inf: \
     g-<b>NW2</b>_nfs ms-drbd_<b>NW2</b>_nfs:Master
   </code></pre>

   La opción `crossmnt` de los recursos del clúster `exportfs` está presente en nuestra documentación para la compatibilidad con versiones anteriores de SLES.  

1. **[1]** Deshabilite el modo de mantenimiento.
   
   <pre><code>sudo crm configure property maintenance-mode=false
   </code></pre>

## <a name="next-steps"></a>Pasos siguientes

* [Instalación de SAP ASCS y la base de datos](high-availability-guide-suse.md)
* [Planeamiento e implementación de Azure Virtual Machines para SAP][planning-guide]
* [Implementación de Azure Virtual Machines para SAP][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP][dbms-guide]
* Para más información sobre cómo establecer la alta disponibilidad y planear la recuperación ante desastres de SAP HANA en Azure Virtual Machines, consulte [Alta disponibilidad de SAP HANA en las máquinas virtuales de Azure][sap-hana-ha].
