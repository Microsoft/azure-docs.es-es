---
title: Solución de problemas de escalabilidad horizontal de SAP HANA con escenarios de HSR y Pacemaker con SLES en VM de Azure | Microsoft Docs
description: Guía para comprobar y solucionar problemas de una configuración compleja de alta disponibilidad de escalabilidad horizontal de SAP HANA basada en SAP HANA System Replication (HSR) y Pacemaker en SLES 12 SP3 que se ejecutan en máquinas virtuales de Azure
services: virtual-machines-linux
documentationcenter: ''
author: hermanndms
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/24/2018
ms.author: hermannd
ms.openlocfilehash: 5c3a24bc9d754a15a0b372667fbcd689365a9aec
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088315"
---
# <a name="verify-and-troubleshoot-sap-hana-scale-out-high-availability-setup-on-sles-12-sp3"></a>Verificación y solución de problemas de configuración de alta disponibilidad con escalabilidad horizontal de SAP HANA en SLES 12 SP3 

[sles-pacemaker-ha-guide]:high-availability-guide-suse-pacemaker.md
[sles-hana-scale-out-ha-paper]:https://www.suse.com/documentation/suse-best-practices/singlehtml/SLES4SAP-hana-scaleOut-PerfOpt-12/SLES4SAP-hana-scaleOut-PerfOpt-12.html
[sap-hana-iaas-list]:https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html
[suse-pacemaker-support-log-files]:https://www.suse.com/support/kb/doc/?id=7022702
[azure-linux-multiple-nics]:https://docs.microsoft.com/azure/virtual-machines/linux/multiple-nics
[suse-cloud-netconfig]:https://www.suse.com/c/multi-nic-cloud-netconfig-ec2-azure/
[sap-list-port-numbers]:https://help.sap.com/viewer/ports
[sles-12-ha-paper]:https://www.suse.com/documentation/sle-ha-12/pdfdoc/book_sleha/book_sleha.pdf
[sles-zero-downtime-paper]:https://www.suse.com/media/presentation/TUT90846_towards_zero_downtime%20_how_to_maintain_sap_hana_system_replication_clusters.pdf
[sap-nw-ha-guide-sles]:high-availability-guide-suse.md
[sles-12-for-sap]:https://www.suse.com/media/white-paper/suse_linux_enterprise_server_for_sap_applications_12_sp1.pdf


Este artículo ayuda a comprobar la configuración del clúster de Pacemaker para SAP HANA con escalabilidad horizontal que se ejecuta en máquinas virtuales de Azure. El clúster se configuró al combinar SAP HANA System Replication (HSR) y SAPHanaSR-ScaleOut del paquete SUSE RPM. Todas las pruebas se realizaron únicamente en SUSE SLES 12 SP3. En las distintas secciones del artículo se abordan distintas áreas y se incluyen comandos de ejemplo y citas de los archivos de configuración. Estos ejemplos se recomiendan como método para comprobar toda la configuración del clúster.



## <a name="important-notes"></a>Notas importantes

Todas las pruebas de SAP HANA de escalabilidad horizontal en combinación con SAP HANA System Replication y Pacemaker se realizaron únicamente con SAP HANA 2.0. La versión del sistema operativo fue SUSE Linux Enterprise Server 12 SP3 for SAP Applications. Se usó la versión más reciente de SAPHanaSR-ScaleOut del paquete RPM de SUSE para configurar el clúster de Pacemaker.
SUSE ha publicado una [descripción detallada de esta configuración optimizada para el rendimiento][sles-hana-scale-out-ha-paper].

Para conocer los tipos de máquina virtual que admiten el escalado horizontal de SAP HANA, consulte el [directorio de IaaS certificado para SAP HANA][sap-hana-iaas-list].

Se produjo un problema técnico con el escalado horizontal de SAP HANA en combinación con varias subredes y NIC virtuales, así como al configurar HSR. Es obligatorio usar las últimas revisiones de SAP HANA 2.0 donde este problema está solucionado. Se admiten las siguientes versiones de SAP HANA: 

* rev2.00.024.04 o superior 
* rev2.00.032 o superior

Si necesita soporte técnico por parte de SUSE, siga esta [guía][suse-pacemaker-support-log-files]. Recopile toda la información sobre el clúster de alta disponibilidad de SAP HANA como se describe en el artículo. El servicio técnico de SUSE necesita esta información para profundizar en el análisis.

Durante las pruebas internas, la configuración del clúster se confundió con un apagado de máquina virtual correcto normal mediante Azure Portal. Por lo tanto, se recomienda probar la conmutación por error del clúster por otros métodos. Use métodos como forzar una alerta de pánico, apagar las redes o migrar el recurso **msl**. Vea los detalles en las secciones siguientes. Se presupone que un apagado estándar se produce a propósito. El mejor ejemplo de un apagado intencionado es por motivos de mantenimiento. Consulte los detalles en [Mantenimiento planeado](#planned-maintenance).

Además, durante las pruebas internas la configuración del clúster se confundió tras una adquisición manual de SAP HANA con el clúster en modo de mantenimiento. Se recomienda cambiarlo de nuevo manualmente antes de finalizar el modo de mantenimiento del clúster. Otra opción consiste en desencadenar una conmutación por error antes de poner el clúster en modo de mantenimiento. Para obtener más información, consulte [Mantenimiento planeado](#planned-maintenance). En la documentación de SUSE se describe la manera de restablecer el clúster en este aspecto mediante el comando **crm**. Pero el enfoque mencionado antes parecía sólido durante las pruebas internas y nunca han aparecido efectos secundarios inesperados.

Cuando se usa el comando **crm migrate**, asegúrese de limpiar la configuración del clúster. Agrega restricciones de ubicación que quizá desconocía. Estas restricciones afectan el comportamiento del clúster. Consulte más detalles en [Mantenimiento planeado](#planned-maintenance).



## <a name="test-system-description"></a>Descripción del sistema de prueba

 Para la certificación y comprobación de la alta disponibilidad de SAP HANA con escalabilidad horizontal, se usó una configuración. Constaba de dos sistemas con tres nodos de SAP HANA: uno maestro y dos trabajos. En la siguiente tabla se enumeran los nombres de máquina virtual y las direcciones IP internas. Todos los ejemplos de comprobación siguientes se realizaron en estas máquinas virtuales. Estos nombres de máquina virtual y direcciones IP de los ejemplos de comandos deberían ayudar a comprender mejor los comandos y sus salidas:


| Tipo de nodo | Nombre de la máquina virtual | Dirección IP |
| --- | --- | --- |
| Nodo maestro en el sitio 1 | hso-hana-vm-s1-0 | 10.0.0.30 |
| Nodo de trabajo 1 en el sitio 1 | hso-hana-vm-s1-1 | 10.0.0.31 |
| Nodo de trabajo 2 en el sitio 1 | hso-hana-vm-s1-2 | 10.0.0.32 |
| | | |
| Nodo maestro en el sitio 2 | hso-hana-vm-s2-0 | 10.0.0.40 |
| Nodo de trabajo 1 en el sitio 2 | hso-hana-vm-s2-1 | 10.0.0.41 |
| Nodo de trabajo 2 en el sitio 2 | hso-hana-vm-s2-2  | 10.0.0.42 |
| | | |
| Mayoría de nodo creador | hso-hana-dm | 10.0.0.13 |
| Servidor de dispositivo SBD | hso-hana-sbd | 10.0.0.19 |
| | | |
| Servidor NFS 1 | hso-nfs-vm-0 | 10.0.0.15 |
| Servidor NFS 2 | hso-nfs-vm-1 | 10.0.0.14 |



## <a name="multiple-subnets-and-vnics"></a>Varias subredes y NIC virtuales

Según las recomendaciones de red de SAP HANA, se crearon tres subredes en una red virtual de Azure. SAP HANA con escalabilidad horizontal en Azure debe estar instalado en modo no compartido. Es decir, cada nodo usa volúmenes de disco locales para **/hana/data** y **/hana/log**. Como los nodos solo usan volúmenes de disco locales, no es necesario definir una subred independiente para el almacenamiento:

- 10.0.2.0/24 para la comunicación entre nodos de SAP HANA
- 10.0.1.0/24 para System Replication HSR de SAP HANA
- 10.0.0.0/24 para todo lo demás

Para obtener información sobre la configuración de SAP HANA relativa al uso de varias redes, consulte [SAP HANA global.ini](#sap-hana-globalini).

Las máquinas virtuales del clúster tienen tres NIC virtuales que se corresponden con el número de subredes. En [Cómo crear una máquina virtual Linux en Azure con red varias tarjetas de interfaz de red][azure-linux-multiple-nics] se describe un posible problema de enrutamiento en Azure al implementar una máquina virtual Linux. Este artículo sobre enrutamiento se aplica solo al uso de varias NIC virtuales. El problema se resuelve en SUSE de forma predeterminada en SLES 12 SP3. Para más información, consulte [Varias NIC con cloud-netconfig en EC2 y Azure][suse-cloud-netconfig].


Para comprobar que SAP HANA está configurado correctamente para utilizar varias redes, ejecute los siguientes comandos. El primer paso es comprobar de nuevo en el nivel de sistema operativo que las tres direcciones IP internas de las tres subredes estén activas. En caso de que haya definido las subredes con distintos intervalos de direcciones IP, debe adaptar los comandos:

<pre><code>
ifconfig | grep "inet addr:10\."
</code></pre>

A continuación, se muestra una salida de ejemplo del segundo nodo de trabajo del sitio 2. Puede ver tres direcciones IP internas distintas, de eth0, eth1 y eth2:

<pre><code>
inet addr:10.0.0.42  Bcast:10.0.0.255  Mask:255.255.255.0
inet addr:10.0.1.42  Bcast:10.0.1.255  Mask:255.255.255.0
inet addr:10.0.2.42  Bcast:10.0.2.255  Mask:255.255.255.0
</code></pre>


A continuación, compruebe los puertos de SAP HANA para el servidor de nombres y HSR. SAP HANA debe escuchar en las subredes correspondientes. Deberá adaptar los comandos en función del número de instancia de SAP HANA. Para el sistema de prueba, el número de instancia era **00**. Hay diferentes formas de averiguar qué puertos se usan. 

La siguiente instrucción SQL devuelve el identificador de instancia, el número de instancia y otra información:

<pre><code>
select * from "SYS"."M_SYSTEM_OVERVIEW"
</code></pre>

Para buscar los números de puerto correctos, puede buscar, por ejemplo, en HANA Studio en **Configuration** (Configuración) o mediante una instrucción SQL:

<pre><code>
select * from M_INIFILE_CONTENTS WHERE KEY LIKE 'listen%'
</code></pre>

Para buscar todos los puertos que se utilizan en la pila de software SAP, incluido SAP HANA, busque [TCP/IP ports of all SAP products][sap-list-port-numbers] (Puertos TCP/IP de todos los productos de SAP).

Dado el número de instancia **00** en el sistema de prueba de SAP HANA 2.0, el número de puerto del servidor de nombres es **30001**. El número de puerto para la comunicación de metadatos de HSR es **40002**. Una opción es iniciar sesión un nodo de trabajo y comprobar los servicios del nodo maestro. En este artículo, la comprobación se realizó en el nodo de trabajo 2 del sitio 2 para intentar conectar con el nodo maestro del sitio 2.

Compruebe el puerto del servidor de nombres:

<pre><code>
nc -vz 10.0.0.40 30001
nc -vz 10.0.1.40 30001
nc -vz 10.0.2.40 30001
</code></pre>

El resultado debe parecerse a la salida de ejemplo siguiente para que se demuestre que para la comunicación entre nodos se usa la subred **10.0.2.0/24**.
Solo la conexión a través de la subred **10.0.2.0/24** debe ser correcta:

<pre><code>
nc: connect to 10.0.0.40 port 30001 (tcp) failed: Connection refused
nc: connect to 10.0.1.40 port 30001 (tcp) failed: Connection refused
Connection to 10.0.2.40 30001 port [tcp/pago-services1] succeeded!
</code></pre>

Ahora compruebe el puerto de HSR **40002**:

<pre><code>
nc -vz 10.0.0.40 40002
nc -vz 10.0.1.40 40002
nc -vz 10.0.2.40 40002
</code></pre>

El resultado debe parecerse a la salida de ejemplo siguiente para que se demuestre que para la comunicación con HSR se usa la subred **10.0.1.0/24**.
Solo la conexión a través de la subred **10.0.1.0/24** debe ser correcta:

<pre><code>
nc: connect to 10.0.0.40 port 40002 (tcp) failed: Connection refused
Connection to 10.0.1.40 40002 port [tcp/*] succeeded!
nc: connect to 10.0.2.40 port 40002 (tcp) failed: Connection refused
</code></pre>



## <a name="corosync"></a>Corosync


El archivo de configuración **corosync** tiene que ser correcto en todos los nodos del clúster, incluido el nodo creador mayoritario. En caso de que la combinación de clúster de un nodo no funcionara según lo previsto, cree o copie **/etc/corosync/corosync.conf** manualmente en todos los nodos y reinicie el servicio. 

Este es el contenido de **corosync.conf** del sistema de prueba a modo de ejemplo.

La primera sección es **totem**, tal como se describe en el paso 11 de la sección [Instalación del clúster](./high-availability-guide-suse-pacemaker.md#cluster-installation). Puede omitir el valor de **mcastaddr**. Simplemente, conserve la entrada existente. Las entradas de **token** y **consensus** deben establecerse según la [documentación de SAP HANA de Microsoft Azure][sles-pacemaker-ha-guide].

<pre><code>
totem {
    version:    2
    secauth:    on
    crypto_hash:    sha1
    crypto_cipher:  aes256
    cluster_name:   hacluster
    clear_node_high_bit: yes

    token:      30000
    token_retransmits_before_loss_const: 10
    join:       60
    consensus:  36000
    max_messages:   20

    interface {
        ringnumber:     0
        bindnetaddr: 10.0.0.0
        mcastaddr:  239.170.19.232
        mcastport:  5405

        ttl:        1
    }
    transport:      udpu

}
</code></pre>

En la segunda sección, **logging**, no se han cambiado los valores predeterminados:

<pre><code>
logging {
    fileline:   off
    to_stderr:  no
    to_logfile:     no
    logfile:    /var/log/cluster/corosync.log
    to_syslog:  yes
    debug:      off
    timestamp:  on
    logger_subsys {
        subsys:     QUORUM
        debug:  off
    }
}
</code></pre>

En la tercera sección aparece **nodelist**. Todos los nodos del clúster tienen que mostrarse con el **identificador de nodo**:

<pre><code>
nodelist {
  node {
   ring0_addr:hso-hana-vm-s1-0
   nodeid: 1
   }
  node {
   ring0_addr:hso-hana-vm-s1-1
   nodeid: 2
   }
  node {
   ring0_addr:hso-hana-vm-s1-2
   nodeid: 3
   }
  node {
   ring0_addr:hso-hana-vm-s2-0
   nodeid: 4
   }
  node {
   ring0_addr:hso-hana-vm-s2-1
   nodeid: 5
   }
  node {
   ring0_addr:hso-hana-vm-s2-2
   nodeid: 6
   }
  node {
   ring0_addr:hso-hana-dm
   nodeid: 7
   }
}
</code></pre>

En la última sección, **quorum**, es importante establecer el valor de **expected_votes** correctamente. Debe ser el número de nodos, mayoría de nodo creador incluida. Y el valor de **two_node** tiene que ser **0**. No quite la entrada por completo. Basta con establecer el valor en **0**.

<pre><code>
quorum {
    # Enable and configure quorum subsystem (default: off)
    # see also corosync.conf.5 and votequorum.5
    provider: corosync_votequorum
    expected_votes: 7
    two_node: 0
}
</code></pre>


Reinicie el servicio mediante **systemctl**:

<pre><code>
systemctl restart corosync
</code></pre>




## <a name="sbd-device"></a>Dispositivo SBD

La configuración de un dispositivo SBD en una máquina virtual de Azure se describe en la sección [Vallado de SBD](./high-availability-guide-suse-pacemaker.md#sbd-fencing).

Primero compruebe si en la máquina virtual del servidor SBD existen entradas de lista de control de acceso para todos los nodos del clúster. En la máquina virtual del servidor SBD, ejecute el siguiente comando:


<pre><code>
targetcli ls
</code></pre>


En el sistema de prueba, la salida del comando es similar al siguiente ejemplo. Los nombres de la lista de control de acceso de tipo **iqn.2006-04.hso-db-0.local:hso-db-0** deben especificarse como el nombre del iniciador correspondiente en las máquinas virtuales. Cada máquina virtual necesita un distinto.

<pre><code>
 | | o- sbddbhso ................................................................... [/sbd/sbddbhso (50.0MiB) write-thru activated]
  | |   o- alua ................................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 1]
  | o- iqn.2006-04.dbhso.local:dbhso ..................................................................................... [TPGs: 1]
  |   o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  |     o- acls .......................................................................................................... [ACLs: 7]
  |     | o- iqn.2006-04.hso-db-0.local:hso-db-0 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-1.local:hso-db-1 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-2.local:hso-db-2 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-3.local:hso-db-3 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-4.local:hso-db-4 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-5.local:hso-db-5 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-6.local:hso-db-6 .................................................................. [Mapped LUNs: 1]
</code></pre>

Después, compruebe que los nombres de iniciador de todas las máquinas virtuales sean diferentes y se correspondan con las entradas mostradas anteriormente. Este es un ejemplo de nodo de trabajo 1 del sitio 1:

<pre><code>
cat /etc/iscsi/initiatorname.iscsi
</code></pre>

La salida tendrá un aspecto similar al siguiente ejemplo:

<pre><code>
##
## /etc/iscsi/iscsi.initiatorname
##
## Default iSCSI Initiatorname.
##
## DO NOT EDIT OR REMOVE THIS FILE!
## If you remove this file, the iSCSI daemon will not start.
## If you change the InitiatorName, existing access control lists
## may reject this initiator.  The InitiatorName must be unique
## for each iSCSI initiator.  Do NOT duplicate iSCSI InitiatorNames.
InitiatorName=iqn.2006-04.hso-db-1.local:hso-db-1
</code></pre>

A continuación, compruebe si **discovery** funciona correctamente. Ejecute el siguiente comando en cada nodo del clúster con la dirección IP de la máquina virtual del servidor SBD:

<pre><code>
iscsiadm -m discovery --type=st --portal=10.0.0.19:3260
</code></pre>

La salida debe tener un aspecto similar al siguiente ejemplo:

<pre><code>
10.0.0.19:3260,1 iqn.2006-04.dbhso.local:dbhso
</code></pre>

El siguiente punto de comprobación es verificar que el nodo ve el dispositivo SDB. Compruébelo en cada nodo, mayoría de nodo creador incluida:

<pre><code>
lsscsi | grep dbhso
</code></pre>

La salida debe tener un aspecto similar al siguiente ejemplo. Sin embargo, los nombres podrían diferir. También podría cambiar el nombre del dispositivo después de reiniciar la máquina virtual:

<pre><code>
[6:0:0:0]    disk    LIO-ORG  sbddbhso         4.0   /dev/sdm
</code></pre>

Dependiendo del estado del sistema, a veces puede ser útil reiniciar los servicios de iSCSI para solucionar problemas. Luego, ejecute los siguientes comandos:

<pre><code>
systemctl restart iscsi
systemctl restart iscsid
</code></pre>


Desde cualquier nodo, puede comprobar si todos los nodos están **clear**. Asegúrese de usar el nombre de dispositivo correcto en un nodo específico:

<pre><code>
sbd -d /dev/sdm list
</code></pre>

La salida debe mostrar **clear** en todos los nodos del clúster:

<pre><code>
0       hso-hana-vm-s1-0        clear
1       hso-hana-vm-s2-2        clear
2       hso-hana-vm-s2-1        clear
3       hso-hana-dm     clear
4       hso-hana-vm-s1-1        clear
5       hso-hana-vm-s2-0        clear
6       hso-hana-vm-s1-2        clear
</code></pre>


Otra comprobación de SBD es la opción **dump** del comando **sbd**. Este es un comando de ejemplo y la salida del nodo creador mayoritario donde el nombre de dispositivo no era **sdm**, sino **sdd**:

<pre><code>
sbd -d /dev/sdd dump
</code></pre>

La salida (excepto el nombre del dispositivo) debe ser el mismo en todos los nodos:

<pre><code>
==Dumping header on disk /dev/sdd
Header version     : 2.1
UUID               : 9fa6cc49-c294-4f1e-9527-c973f4d5a5b0
Number of slots    : 255
Sector size        : 512
Timeout (watchdog) : 60
Timeout (allocate) : 2
Timeout (loop)     : 1
Timeout (msgwait)  : 120
==Header on disk /dev/sdd is dumped
</code></pre>

Una comprobación más de SBD consiste en poder enviar un mensaje a otro nodo. Ejecute el siguiente comando en el nodo de trabajo 1 del sitio 2 para enviar un mensaje al nodo de trabajo 2 del sitio 2:

<pre><code>
sbd -d /dev/sdm message hso-hana-vm-s2-2 test
</code></pre>

En la máquina virtual de destino, que era **hso-hana-vm-s2-2** en este ejemplo, puede encontrar la siguiente entrada en **/var/log/messages**:

<pre><code>
/dev/disk/by-id/scsi-36001405e614138d4ec64da09e91aea68:   notice: servant: Received command test from hso-hana-vm-s2-1 on disk /dev/disk/by-id/scsi-36001405e614138d4ec64da09e91aea68
</code></pre>

Compruebe que las entradas de **/etc/sysconfig/sbd** se corresponden con la descripción de [Configuración de Pacemaker en SUSE Linux Enterprise Server en Azure](./high-availability-guide-suse-pacemaker.md#sbd-fencing). Verifique que la configuración de inicio en **/etc/iscsi/iscsid.conf** está establecida en automático.

Las entradas siguientes son importantes en **/etc/sysconfig/sbd**. Adapte el valor de **id** si es necesario:

<pre><code>
SBD_DEVICE="/dev/disk/by-id/scsi-36001405e614138d4ec64da09e91aea68;"
SBD_PACEMAKER=yes
SBD_STARTMODE=always
SBD_WATCHDOG=yes
</code></pre>


Compruebe la configuración de inicio en **/etc/iscsi/iscsid.conf**. Con el comando **iscsiadm** debería aparecer la configuración necesaria, que se describe en la documentación. Compruébela y adáptela manualmente con **vi** si es diferente.

Este comando establece el comportamiento de inicio:

<pre><code>
iscsiadm -m node --op=update --name=node.startup --value=automatic
</code></pre>

Cree esta entrada en **/etc/iscsi/iscsid.conf**:

<pre><code>
node.startup = automatic
</code></pre>

Durante las pruebas y la verificación, después de reiniciar una máquina virtual, el dispositivo SBD dejaba de ser visible en algunos casos. Se produjo una discrepancia entre la configuración de inicio y lo que mostraba YaST2. Para comprobar la configuración, siga estos pasos:

1. Inicie YaST2.
2. Seleccione **Network Services** en el panel izquierdo.
3. Desplácese hacia abajo en el lado derecho a **iSCSI Initiator** (Iniciador de iSCI) y selecciónelo.
4. En la siguiente pantalla, en la pestaña **Service** (Servicio), debe ver el nombre de iniciador único del nodo.
5. Encima del nombre del iniciador, asegúrese de que el valor **Service Start** (Iniciar servicio) está establecido en **When Booting** (Al arrancar).
6. Si no es así, establézcalo en **When Booting** (Al arrancar) en lugar de **Manually** (Manualmente).
7. A continuación, cambie la pestaña superior a **Connected Targets** (Destinos conectados).
8. En la pantalla de **destinos conectados** verá una entrada para el dispositivo SBD como en este ejemplo: **10.0.0.19:3260 iqn.2006-04.dbhso.local:dbhso**.
9. Compruebe si el valor de **inicio** está establecido en **onboot**.
10. Si no, elija **Edit** (Editar) y cámbielo.
11. Guarde los cambios y salga de YaST2.



## <a name="pacemaker"></a>Pacemaker

Con todo configurado correctamente, puede ejecutar el siguiente comando en los nodos para comprobar el estado del servicio Pacemaker:

<pre><code>
systemctl status pacemaker
</code></pre>

La parte superior de la salida debe parecerse al siguiente ejemplo. Es importante que el estado después de **Active** se muestre como **loaded** y **active (running)** . El estado después de que se debe mostrar **Loaded** como **enabled**.

<pre><code>
  pacemaker.service - Pacemaker High Availability Cluster Manager
   Loaded: loaded (/usr/lib/systemd/system/pacemaker.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2018-09-07 05:56:27 UTC; 4 days ago
     Docs: man:pacemakerd
           http://clusterlabs.org/doc/en-US/Pacemaker/1.1-pcs/html/Pacemaker_Explained/index.html
 Main PID: 4496 (pacemakerd)
    Tasks: 7 (limit: 4915)
   CGroup: /system.slice/pacemaker.service
           ├─4496 /usr/sbin/pacemakerd -f
           ├─4499 /usr/lib/pacemaker/cib
           ├─4500 /usr/lib/pacemaker/stonithd
           ├─4501 /usr/lib/pacemaker/lrmd
           ├─4502 /usr/lib/pacemaker/attrd
           ├─4503 /usr/lib/pacemaker/pengine
           └─4504 /usr/lib/pacemaker/crmd
</code></pre>

En caso de que la configuración continúe como **disabled**, ejecute el siguiente comando:

<pre><code>
systemctl enable pacemaker
</code></pre>

Para ver todos los recursos configurados en Pacemaker, ejecute el siguiente comando:

<pre><code>
crm status
</code></pre>

La salida debe tener un aspecto similar al siguiente ejemplo. No importa que los recursos **cln** y **msl** aparezcan como detenidos en la mayoría de las máquinas virtuales creadoras (**hso-hana-dm**). No hay ninguna instalación de SAP HANA en la mayoría de nodo creador. Por lo tanto, los recursos **cln** y **msl** se muestren como detenidos. Es importante que se muestre el total correcto de máquinas virtuales (**7**). Todas las máquinas virtuales que forman parte del clúster deben aparecer con estado **Online** (En línea). El nodo maestro principal actual se debe reconocer correctamente. En este ejemplo es **hso-hana-vm-s1-0**:

<pre><code>
Stack: corosync
Current DC: hso-hana-dm (version 1.1.16-4.8-77ea74d) - partition with quorum
Last updated: Tue Sep 11 15:56:40 2018
Last change: Tue Sep 11 15:56:23 2018 by root via crm_attribute on hso-hana-vm-s1-0

7 nodes configured
17 resources configured

Online: [ hso-hana-dm hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started hso-hana-dm
 Clone Set: cln_SAPHanaTop_HSO_HDB00 [rsc_SAPHanaTop_HSO_HDB00]
     Started: [ hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm ]
 Master/Slave Set: msl_SAPHanaCon_HSO_HDB00 [rsc_SAPHanaCon_HSO_HDB00]
     Masters: [ hso-hana-vm-s1-0 ]
     Slaves: [ hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm ]
 Resource Group: g_ip_HSO_HDB00
     rsc_ip_HSO_HDB00   (ocf::heartbeat:IPaddr2):       Started hso-hana-vm-s1-0
     rsc_nc_HSO_HDB00   (ocf::heartbeat:anything):      Started hso-hana-vm-s1-0
</code></pre>

Una característica importante de Pacemaker es su modo de mantenimiento. En este modo, puede realizar modificaciones sin provocar una acción inmediata del clúster. Un ejemplo es un reinicio de máquina virtual. Un caso de uso típico sería el mantenimiento planeado de la infraestructura de Azure o del sistema operativo. Consulte [Mantenimiento planeado](#planned-maintenance). Use el siguiente comando para que Pacemaker pase a modo de mantenimiento:

<pre><code>
crm configure property maintenance-mode=true
</code></pre>

Al realizar comprobaciones con **crm estado**, observe que en la salida todos los recursos están marcados como **unmanaged** (no administrados). En este estado, el clúster no reacciona a los cambios del tipo inicio y detención de SAP HANA.
A continuación, se muestra una salida de ejemplo del comando **crm status** con el clúster en modo de mantenimiento:

<pre><code>
Stack: corosync
Current DC: hso-hana-dm (version 1.1.16-4.8-77ea74d) - partition with quorum
Last updated: Wed Sep 12 07:48:10 2018
Last change: Wed Sep 12 07:46:54 2018 by root via cibadmin on hso-hana-vm-s2-1

7 nodes configured
17 resources configured

              *** Resource management is DISABLED ***
  The cluster will not attempt to start, stop or recover services

Online: [ hso-hana-dm hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started hso-hana-dm (unmanaged)
 Clone Set: cln_SAPHanaTop_HSO_HDB00 [rsc_SAPHanaTop_HSO_HDB00] (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s1-1 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s1-0 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s1-2 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s2-1 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s2-2 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s2-0 (unmanaged)
     Stopped: [ hso-hana-dm ]
 Master/Slave Set: msl_SAPHanaCon_HSO_HDB00 [rsc_SAPHanaCon_HSO_HDB00] (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s1-1 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s1-2 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s2-1 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s2-2 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Master hso-hana-vm-s2-0 (unmanaged)
     Stopped: [ hso-hana-dm hso-hana-vm-s1-0 ]
 Resource Group: g_ip_HSO_HDB00
     rsc_ip_HSO_HDB00   (ocf::heartbeat:IPaddr2):       Started hso-hana-vm-s2-0 (unmanaged)
     rsc_nc_HSO_HDB00   (ocf::heartbeat:anything):      Started hso-hana-vm-s2-0 (unmanaged)
</code></pre>


Este comando de ejemplo se muestra cómo finalizar el modo de mantenimiento del clúster:

<pre><code>
crm configure property maintenance-mode=false
</code></pre>


Otro comando **crm** obtiene la configuración del clúster completa en un editor, por lo que se puede editar. Después de guardar los cambios, el clúster inicia las acciones apropiadas:

<pre><code>
crm configure edit
</code></pre>

Para ver simplemente la configuración completa del clúster, use la opción **show** de crm:

<pre><code>
crm configure show
</code></pre>



Tras errores en los recursos de clúster, el comando **crm status** muestra una lista **Failed Actions**. Verá la salida de ejemplo siguiente:


<pre><code>
Stack: corosync
Current DC: hso-hana-dm (version 1.1.16-4.8-77ea74d) - partition with quorum
Last updated: Thu Sep 13 07:30:44 2018
Last change: Thu Sep 13 07:30:20 2018 by root via crm_attribute on hso-hana-vm-s1-0

7 nodes configured
17 resources configured

Online: [ hso-hana-dm hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started hso-hana-dm
 Clone Set: cln_SAPHanaTop_HSO_HDB00 [rsc_SAPHanaTop_HSO_HDB00]
     Started: [ hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm ]
 Master/Slave Set: msl_SAPHanaCon_HSO_HDB00 [rsc_SAPHanaCon_HSO_HDB00]
     Masters: [ hso-hana-vm-s1-0 ]
     Slaves: [ hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm hso-hana-vm-s2-0 ]
 Resource Group: g_ip_HSO_HDB00
     rsc_ip_HSO_HDB00   (ocf::heartbeat:IPaddr2):       Started hso-hana-vm-s1-0
     rsc_nc_HSO_HDB00   (ocf::heartbeat:anything):      Started hso-hana-vm-s1-0

Failed Actions:
* rsc_SAPHanaCon_HSO_HDB00_monitor_60000 on hso-hana-vm-s2-0 'unknown error' (1): call=86, status=complete, exitreason='none',
    last-rc-change='Wed Sep 12 17:01:28 2018', queued=0ms, exec=277663ms
</code></pre>

Tras los errores es necesario limpiar el clúster. Simplemente use el comando **crm** de nuevo y la opción del comando **cleanup** para deshacerse de estas entradas de acción errónea. Indique el nombre del recurso del clúster correspondiente como sigue:

<pre><code>
crm resource cleanup rsc_SAPHanaCon_HSO_HDB00
</code></pre>

El comando debe devolver una salida similar a la del siguiente ejemplo:

<pre><code>
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-dm
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s1-0
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s1-1
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s1-2
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s2-0
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s2-1
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s2-2
Waiting for 7 replies from the CRMd....... OK
</code></pre>



## <a name="failover-or-takeover"></a>Conmutación por error o adquisición

Como ya se mencionó en [Notas importantes](#important-notes), no debe usar el apagado estable estándar para probar la conmutación por error de clúster ni la adquisición de HSR de SAP HANA. En su lugar, se recomienda que desencadene una alerta de pánico de kernel, fuerce una migración de recursos o quizás apague todas las redes en el nivel de sistema operativo de una máquina virtual. Otro método sería el comando **crm \<node\> standby**. Consulte el [documento de SUSE][sles-12-ha-paper]. 

A continuación, se encuentran tres comandos de ejemplo para forzar una conmutación por error de clúster:

<pre><code>
echo c &gt /proc/sysrq-trigger

crm resource migrate msl_SAPHanaCon_HSO_HDB00 hso-hana-vm-s2-0 force

wicked ifdown eth0
wicked ifdown eth1
wicked ifdown eth2
......
wicked ifdown eth&ltn&gt
</code></pre>

Como se describe en [Mantenimiento planeado](#planned-maintenance), una buena forma de supervisar las actividades del clúster es ejecutar **SAPHanaSR showAttr** con el comando **watch**:

<pre><code>
watch SAPHanaSR-showAttr
</code></pre>

Además, ayuda a examinar el estado general de SAP HANA a partir de un script de Python de SAP. La configuración del clúster busca este valor de estado. Esto se clarifica al pensar en un error de nodo de trabajo. Si este deja de funcionar, SAP HANA no devuelve inmediatamente un error de estado de todo el sistema de escalabilidad horizontal. 

Se pueden realizar varios reintentos para evitar conmutaciones por error innecesarias. El clúster reacciona solo si el estado cambia de **correcto** (valor devuelto de **4**) a **error** (valor devuelto de **1**). Por lo tanto, es correcto si la salida de **SAPHanaSR showAttr** muestra una máquina virtual con el estado **offline** (sin conexión). Pero no hay ninguna actividad aún para cambiar el principal y el secundario. No se desencadena actividad en el clúster hasta que SAP HANA devuelva un error.

Puede supervisar el estado general de SAP HANA como usuario **\<HANA SID\>adm** mediante una llamada al script de Python se SAP de esta manera. Es posible que deba adaptar la ruta de acceso:

<pre><code>
watch python /hana/shared/HSO/exe/linuxx86_64/HDB_2.00.032.00.1533114046_eeaf4723ec52ed3935ae0dc9769c9411ed73fec5/python_support/landscapeHostConfiguration.py
</code></pre>

La salida de este comando debe tener el aspecto del siguiente ejemplo. Es importante la columna **Host Status** columna, así como **overall host status**. De hecho, la salida real es más amplia, con columnas adicionales.
Para que la tabla de salida gane legibilidad en este documento, se ha eliminado la mayoría de las columnas del lado derecho:

<pre><code>
| Host             | Host   | Host   | Failover | Remove | 
|                  | Active | Status | Status   | Status | 
|                  |        |        |          |        | 
| ---------------- | ------ | ------ | -------- | ------ |    .......
| hso-hana-vm-s2-0 | yes    | ok     |          |        |        
| hso-hana-vm-s2-1 | yes    | ok     |          |        |         
| hso-hana-vm-s2-2 | yes    | ok     |          |        |        

overall host status: ok
</code></pre>


Hay otro comando para comprobar las actividades actuales del clúster. Consulte el siguiente comando y el final de la salida tras la eliminación del nodo maestro del sitio principal. Puede ver la lista de acciones de transición, como la **promoción** del antiguo nodo maestro secundario (**hso-hana-vm-s2-0**) al nuevo maestro principal. Si todo es correcto y todas las actividades han finalizado, la lista **Transition Summary** debe estar vacía.

<pre><code>
 crm_simulate -Ls

...........

Transition Summary:
 * Fence hso-hana-vm-s1-0
 * Stop    rsc_SAPHanaTop_HSO_HDB00:1   (hso-hana-vm-s1-0)
 * Demote  rsc_SAPHanaCon_HSO_HDB00:1   (Master -> Stopped hso-hana-vm-s1-0)
 * Promote rsc_SAPHanaCon_HSO_HDB00:5   (Slave -> Master hso-hana-vm-s2-0)
 * Move    rsc_ip_HSO_HDB00     (Started hso-hana-vm-s1-0 -> hso-hana-vm-s2-0)
 * Move    rsc_nc_HSO_HDB00     (Started hso-hana-vm-s1-0 -> hso-hana-vm-s2-0)
</code></pre>



## <a name="planned-maintenance"></a>Mantenimiento planeado 

Hay distintos casos de uso en cuanto a mantenimiento planeado. Una de las consideraciones es, por ejemplo, si se trata solo de mantenimiento de la infraestructura, como cambios a nivel de sistema operativo y configuración de disco, o de una actualización de HANA.
Puede encontrar información adicional en los documentos de SUSE como [Hacia el tiempo de inactividad cero][sles-zero-downtime-paper] o [ Escenario optimizado para el rendimiento de SAP HANA SR][sles-12-for-sap]. Estos documentos también incluyen ejemplos de cómo migrar manualmente un sitio principal.

Se han realizado pruebas internas minuciosas para verificar el caso de uso de mantenimiento de la infraestructura. Para evitar cualquier tipo de problema relacionado con la migración de la réplica principal, se decidió migrarla siempre antes de que el clúster pase a modo de mantenimiento. De este modo no es necesario que el clúster olvide la situación anterior (qué lado era principal y qué lado era secundario).

En este sentido, hay dos situaciones diferentes:

- **Mantenimiento planeado en el sitio secundario actual**. En este caso, el clúster puede pasar a modo de mantenimiento y se realiza el trabajo en el sitio secundario sin que ello afecte al clúster.

- **Mantenimiento planeado en el sitio principal actual**. Para que los usuarios pueden seguir trabajando durante el mantenimiento, deberá forzar una conmutación por error. Con este enfoque debe desencadenar la conmutación por error del clúster mediante Pacemaker y no solo a nivel de HSR de SAP HANA. La configuración de Pacemaker desencadena automáticamente la adquisición de SAP HANA. También debe realizar la conmutación por error antes de que el clúster pase a modo de mantenimiento.

El procedimiento de mantenimiento en el sitio secundario actual es el siguiente:

1. Pase a modo de mantenimiento del clúster.
2. Realice el trabajo en el sitio secundario. 
3. Termine el modo de mantenimiento del clúster.

El procedimiento de mantenimiento en el sitio principal actual es más complejo:

1. Desencadene manualmente una conmutación por error o adquisición de SAP HANA mediante la migración de recursos de Pacemaker. Consulte los detalles siguientes.
2. SAP HANA en el sitio principal anterior se apaga a causa de la configuración del clúster.
3. Pase a modo de mantenimiento del clúster.
4. Una vez terminado el mantenimiento, registre el sitio principal anterior como el nuevo sitio secundario.
5. Limpie la configuración del clúster. Consulte los detalles siguientes.
6. Termine el modo de mantenimiento del clúster.


Con la migración de un recurso se agrega una entrada a la configuración del clúster. Por ejemplo, forzar una conmutación por error. Tendrá que eliminar estas entradas antes de finalizar el modo de mantenimiento. Consulte el ejemplo siguiente.

El primer paso es forzar la conmutación por error del clúster mediante la migración del recurso **msl** al nodo maestro secundario actual. Este comando emite una advertencia de que se ha creado una **restricción de movimiento**:

<pre><code>
crm resource migrate msl_SAPHanaCon_HSO_HDB00 force

INFO: Move constraint created for msl_SAPHanaCon_HSO_HDB00
</code></pre>


Compruebe el proceso de conmutación por error con el comando **SAPHanaSR-showAttr**. Para supervisar el estado del clúster es abrir una ventana del shell específica e iniciar el comando con **watch**:

<pre><code>
watch SAPHanaSR-showAttr
</code></pre>

La salida debe reflejar la conmutación por error manual. El nodo maestro secundario anterior se **promovió**, en este ejemplo, **hso-hana-vm-s2-0**. El sitio principal anterior se detuvo; el valor de **lss** **1** del nodo maestro principal anterior, **hso-hana-vm-s1-0**: 

<pre><code>
Global cib-time                 prim  sec srHook sync_state
------------------------------------------------------------
global Wed Sep 12 07:40:02 2018 HSOS2 -   SFAIL  SFAIL


Sites lpt        lss mns              srr
------------------------------------------
HSOS1 10         1   hso-hana-vm-s1-0 P
HSOS2 1536738002 4   hso-hana-vm-s2-0 P


Hosts            clone_state node_state roles                        score  site
----------------------------------------------------------------------------------
hso-hana-dm                  online
hso-hana-vm-s1-0 UNDEFINED   online     master1::worker:             150    HSOS1
hso-hana-vm-s1-1 DEMOTED     online     slave::worker:               -10000 HSOS1
hso-hana-vm-s1-2 DEMOTED     online     slave::worker:               -10000 HSOS1
hso-hana-vm-s2-0 PROMOTED    online     master1:master:worker:master 150    HSOS2
hso-hana-vm-s2-1 DEMOTED     online     slave:slave:worker:slave     -10000 HSOS2
hso-hana-vm-s2-2 DEMOTED     online     slave:slave:worker:slave     -10000 HSOS2
</code></pre>

Después de la conmutación por error del clúster y la adquisición de SAP HANA, pase al modo de mantenimiento del clúster como se describe en [Pacemaker](#pacemaker).

Los comandos **SAPHanaSR-showAttr** y **crm status** no indican nada sobre las restricciones creadas por la migración de recursos. Una opción para ver estas restricciones es mostrar toda la configuración de los recursos del clúster con el siguiente comando:

<pre><code>
crm configure show
</code></pre>

En la configuración del clúster, encontrará una nueva restricción de ubicación provocada por la migración de recursos manual anterior. Esta entrada de ejemplo empieza por **location cli-** :

<pre><code>
location cli-ban-msl_SAPHanaCon_HSO_HDB00-on-hso-hana-vm-s1-0 msl_SAPHanaCon_HSO_HDB00 role=Started -inf: hso-hana-vm-s1-0
</code></pre>

Lamentablemente, estas restricciones podrían afectar al comportamiento general del clúster. Por lo tanto, es obligatorio para quitarlas antes de volver a poner todo el sistema en funcionamiento de nuevo. Con el comando **unmigrate** se eliminan las restricciones de ubicación que se crearon anteriormente. La nomenclatura puede resultar un poco confusa. No intente devolver el recurso de nuevo a la máquina virtual original de la migración. Simplemente elimina las restricciones de ubicación y devuelve la información correspondiente al ejecutar el comando:


<pre><code>
crm resource unmigrate msl_SAPHanaCon_HSO_HDB00

INFO: Removed migration constraints for msl_SAPHanaCon_HSO_HDB00
</code></pre>

Al final del trabajo de mantenimiento, detenga el modo de mantenimiento del clúster tal como se muestra en [Pacemaker](#pacemaker).



## <a name="hb_report-to-collect-log-files"></a>hb_report para recopilar los archivos de registro

Para analizar problemas de clúster de Pacemaker, resulta útil (y el soporte técnico SUSE lo pide) ejecutar la utilidad **hb_report**. Recopila todos los archivos de registro importantes que necesita para analizar qué ocurrió. Esta llamada de ejemplo usa un tiempo de inicio y finalización donde se ha producido un incidente específico. Consulte también [Notas importantes](#important-notes):

<pre><code>
hb_report -f "2018/09/13 07:36" -t "2018/09/13 08:00" /tmp/hb_report_log
</code></pre>

El comando indica dónde puso los archivos de registro comprimidos:

<pre><code>
The report is saved in /tmp/hb_report_log.tar.bz2
Report timespan: 09/13/18 07:36:00 - 09/13/18 08:00:00
</code></pre>

Puede extraer los distintos archivos mediante el comando **tar** estándar:

<pre><code>
tar -xvf hb_report_log.tar.bz2
</code></pre>

Al buscar en los archivos extraídos se encuentran todos los archivos de registro. La mayoría de ellos se han puesto en directorios independientes para cada nodo del clúster:

<pre><code>
-rw-r--r-- 1 root root  13655 Sep 13 09:01 analysis.txt
-rw-r--r-- 1 root root  14422 Sep 13 09:01 description.txt
-rw-r--r-- 1 root root      0 Sep 13 09:01 events.txt
-rw-r--r-- 1 root root 275560 Sep 13 09:00 ha-log.txt
-rw-r--r-- 1 root root     26 Sep 13 09:00 ha-log.txt.info
drwxr-xr-x 4 root root   4096 Sep 13 09:01 hso-hana-dm
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s1-0
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s1-1
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s1-2
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s2-0
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s2-1
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s2-2
-rw-r--r-- 1 root root 264726 Sep 13 09:00 journal.log
</code></pre>


En este intervalo de tiempo se eliminó el nodo maestro **hso-hana-vm-s1-0** actual. En **journal.log** puede encontrar entradas relacionadas con este evento:

<pre><code>
2018-09-13T07:38:01+0000 hso-hana-vm-s2-1 su[93494]: (to hsoadm) root on none
2018-09-13T07:38:01+0000 hso-hana-vm-s2-1 su[93494]: pam_unix(su-l:session): session opened for user hsoadm by (uid=0)
2018-09-13T07:38:01+0000 hso-hana-vm-s2-1 systemd[1]: Started Session c44290 of user hsoadm.
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [TOTEM ] A new membership (10.0.0.13:120996) was formed. Members left: 1
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [TOTEM ] Failed to receive the leave message. failed: 1
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 attrd[28313]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 attrd[28313]:   notice: Removing all hso-hana-vm-s1-0 attributes for peer loss
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 attrd[28313]:   notice: Purged 1 peer with id=1 and/or uname=hso-hana-vm-s1-0 from the membership cache
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 stonith-ng[28311]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 stonith-ng[28311]:   notice: Purged 1 peer with id=1 and/or uname=hso-hana-vm-s1-0 from the membership cache
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 cib[28310]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [QUORUM] Members[6]: 7 2 3 4 5 6
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [MAIN  ] Completed service synchronization, ready to provide service.
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 crmd[28315]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 pacemakerd[28308]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 cib[28310]:   notice: Purged 1 peer with id=1 and/or uname=hso-hana-vm-s1-0 from the membership cache
2018-09-13T07:38:03+0000 hso-hana-vm-s2-1 su[93494]: pam_unix(su-l:session): session closed for user hsoadm
</code></pre>

Otro ejemplo es el archivo de registro de Pacemaker del nodo maestro secundario que se convirtió en el nuevo maestro principal. Este extracto que muestra que el estado del nodo maestro principal eliminado se estableció en **offline**:

<pre><code>
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 3 still member of group stonith-ng (peer=hso-hana-vm-s1-2, counter=5.1)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 4 still member of group stonith-ng (peer=hso-hana-vm-s2-0, counter=5.2)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 5 still member of group stonith-ng (peer=hso-hana-vm-s2-1, counter=5.3)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 6 still member of group stonith-ng (peer=hso-hana-vm-s2-2, counter=5.4)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 7 still member of group stonith-ng (peer=hso-hana-dm, counter=5.5)
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: pcmk_cpg_membership:      Node 1 left group crmd (peer=hso-hana-vm-s1-0, counter=5.0)
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: crm_update_peer_proc:     pcmk_cpg_membership: Node hso-hana-vm-s1-0[1] - corosync-cpg is now offline
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: peer_update_callback:     Client hso-hana-vm-s1-0/peer now has status [offline] (DC=hso-hana-dm, changed=4000000)
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: pcmk_cpg_membership:      Node 2 still member of group crmd (peer=hso-hana-vm-s1-1, counter=5.0)
</code></pre>





## <a name="sap-hana-globalini"></a>global.ini de SAP HANA


Los extractos siguientes provienen del archivo **global.ini** de SAP HANA en el sitio 2 del clúster. Este ejemplo muestra las entradas de resolución de nombres de host para el uso de distintas redes para la comunicación entre nodos de SAP HANA y HSR:

<pre><code>
[communication]
tcp_keepalive_interval = 20
internal_network = 10.0.2/24
listeninterface = .internal
</code></pre>


<pre><code>
[internal_hostname_resolution]
10.0.2.40 = hso-hana-vm-s2-0
10.0.2.42 = hso-hana-vm-s2-2
10.0.2.41 = hso-hana-vm-s2-1
</code></pre>

<pre><code>
[ha_dr_provider_SAPHanaSR]
provider = SAPHanaSR
path = /hana/shared/myHooks
execution_order = 1
</code></pre>

<pre><code>
[system_replication_communication]
listeninterface = .internal

[system_replication_hostname_resolution]
10.0.1.30 = hso-hana-vm-s1-0
10.0.1.31 = hso-hana-vm-s1-1
10.0.1.32 = hso-hana-vm-s1-2
10.0.1.40 = hso-hana-vm-s2-0
10.0.1.41 = hso-hana-vm-s2-1
10.0.1.42 = hso-hana-vm-s2-2
</code></pre>



## <a name="hawk"></a>HAWK

La solución de clúster proporciona una interfaz de explorador que ofrece una interfaz gráfica de usuario para los usuarios que prefieren los menús y los gráficos en comparación con todos los comandos a nivel de shell.
Para usar la interfaz del explorador, reemplace **\<node\>** por un nodo de SAP HANA real en la siguiente dirección URL. A continuación, escriba las credenciales del clúster (usuario **clúster**):

<pre><code>
https://&ltnode&gt:7630
</code></pre>

Esta captura de pantalla muestra el panel del clúster:


![Panel del clúster HAWK](media/hana-vm-scale-out-HA-troubleshooting/hawk-1.png)


Este ejemplo muestra las restricciones de ubicación que provoca la migración de recursos de clúster como se explica en [Mantenimiento planeado](#planned-maintenance):


![Lista de restricciones de HAWK](media/hana-vm-scale-out-HA-troubleshooting/hawk-2.png)


También puede cargar la salida **hb_report** en HAWK, en **History** (Historial), como se indica a continuación. Consulte hb_report para recopilar los archivos de registro: 

![Salida de hb_report de la carga de HAWK](media/hana-vm-scale-out-HA-troubleshooting/hawk-3.png)

Con **History Explorer** (Explorador del historial), puede pasar por todas las transiciones del clúster incluidas en la salida **hb_report**:

![Transiciones de HAWK en la salida hb_report](media/hana-vm-scale-out-HA-troubleshooting/hawk-4.png)

Esta captura de pantalla final muestra la sección **Detalles** de una única transición. El clúster reaccionó en un bloqueo del nodo maestro principal, nodo **hso-hana-vm-s1-0**. Ahora está promocionando el nodo secundario como el nuevo maestro, **hso-hana-vm-s2-0**:

![Transición única de HAWK](media/hana-vm-scale-out-HA-troubleshooting/hawk-5.png)


## <a name="next-steps"></a>Pasos siguientes

Esta guía de solución de problemas describe la alta disponibilidad para SAP HANA en una configuración de escalabilidad horizontal. Además de la base de datos, otro componente importante dentro de un entorno de SAP es la pila de SAP NetWeaver. Obtenga información sobre la [alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure con SUSE Enterprise Linux Server][sap-nw-ha-guide-sles].
