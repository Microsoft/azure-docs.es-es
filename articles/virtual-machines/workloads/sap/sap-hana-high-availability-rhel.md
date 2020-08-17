---
title: Alta disponibilidad de SAP HANA en VM de Azure en RHEL | Microsoft Docs
description: Establezca alta disponibilidad de SAP HANA en máquinas virtuales (VM) de Azure.
services: virtual-machines-linux
documentationcenter: ''
author: rdeltcheva
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/21/2020
ms.author: radeltch
ms.openlocfilehash: 2ce3a4116c12065bbaee8e11d5ada3b8c89b1a9d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088230"
---
# <a name="high-availability-of-sap-hana-on-azure-vms-on-red-hat-enterprise-linux"></a>Alta disponibilidad de SAP HANA en máquinas virtuales de Azure en Red Hat Enterprise Linux

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
[2388694]:https://launchpad.support.sap.com/#/notes/2388694
[2292690]:https://launchpad.support.sap.com/#/notes/2292690
[2455582]:https://launchpad.support.sap.com/#/notes/2455582
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2009879]:https://launchpad.support.sap.com/#/notes/2009879

[sap-swcenter]:https://launchpad.support.sap.com/#/softwarecenter
[template-multisid-db]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json

Para el desarrollo local, puede usar la replicación del sistema de HANA o el almacenamiento compartido para establecer alta disponibilidad para SAP HANA.
En las máquinas virtuales (VM) de Azure, la replicación del sistema de HANA en Azure es actualmente la única función admitida de alta disponibilidad.
La replicación de SAP HANA se realiza con un nodo principal y al menos uno secundario. Los cambios en los datos del nodo principal se replican en los secundarios de forma sincrónica o asincrónica.

En este artículo se describe cómo implementar y configurar las máquinas virtuales, instalar la plataforma del clúster e instalar y configurar la replicación del sistema de SAP HANA.
En las configuraciones de ejemplo, se usan los comandos de instalación, el número de instancia **03** y el identificador del sistema de HANA **HN1**.

Lea primero las notas y los documentos de SAP siguientes:

* Nota de SAP [1928533], que incluye:
  * La lista de tamaños de máquina virtual de Azure que se admiten para la implementación del software de SAP.
  * Información importante sobre capacidad para los tamaños de máquina virtual de Azure.
  * El software de SAP admitido y combinaciones de sistema operativo y base de datos.
  * La versión del kernel de SAP necesaria para Windows y Linux en Microsoft Azure.
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
* [Replicación del sistema de SAP HANA en un clúster de Pacemaker](https://access.redhat.com/articles/3004101)
* Documentación general de RHEL
  * [Introducción al complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [Administración del complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [Referencia del complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
* Documentación de RHEL específica para Azure:
  * [Directivas de compatibilidad para clústeres de alta disponibilidad RHEL: instancias de Microsoft Azure Virtual Machines como miembros del clúster](https://access.redhat.com/articles/3131341)
  * [Instalación y configuración de un clúster de alta disponibilidad de Red Hat Enterprise Linux 7.4 (y versiones posteriores) en Microsoft Azure](https://access.redhat.com/articles/3252491)
  * [Instalación de SAP HANA en Red Hat Enterprise Linux para su uso en Microsoft Azure](https://access.redhat.com/solutions/3193782)

## <a name="overview"></a>Información general

Para lograr una alta disponibilidad, SAP HANA se instala en dos máquinas virtuales. Los datos se replican mediante la replicación del sistema de HANA.

![Información general sobre alta disponibilidad de SAP HANA](./media/sap-hana-high-availability-rhel/ha-hana.png)

En la instalación de la replicación del sistema de SAP HANA se usa un nombre de host virtual dedicado y direcciones IP virtuales. En Azure, se requiere un equilibrador de carga para usar una dirección IP virtual. En la lista siguiente se muestra la configuración del equilibrador de carga:

* Configuración de front-end: Dirección IP 10.0.0.13 para hn1-db
* Configuración de back-end: Se conecta a interfaces de red principales de todas las máquinas virtuales que deben formar parte de la Replicación del sistema de HANA.
* Puerto de sondeo: Puerto 62503
* Reglas de equilibrio de carga: 30313 TCP, 30315 TCP, 30317 TCP, 30340 TCP, 30341 TCP, 30342 TCP

## <a name="deploy-for-linux"></a>Implementación para Linux

Azure Marketplace contiene una imagen de Red Hat Enterprise Linux 7.4 para SAP HANA que puede usar para implementar nuevas máquinas virtuales.

### <a name="deploy-with-a-template"></a>Implementación con una plantilla

Para implementar todos los recursos necesarios, puede usar una de las plantillas de inicio rápido de GitHub. La plantilla implementa las máquinas virtuales, el equilibrador de carga, el conjunto de disponibilidad, etc.
Para implementar la plantilla, siga estos pasos:

1. Abra la [plantilla de base de datos][template-multisid-db] en Azure Portal.
1. Escriba los siguientes parámetros:
    * **Identificador de sistema SAP**: Escriba el identificador del sistema SAP que se va a instalar. El identificador se usa como prefijo de los recursos que se implementan.
    * **Tipo de SO**: Seleccione una de las distribuciones de Linux. En este ejemplo, seleccione **RHEL 7**.
    * **Tipo de base de datos**: Seleccione **HANA**.
    * **Tamaño del sistema SAP**: Escriba la cantidad de SAPS que va a proporcionar el nuevo sistema. Si no está seguro de cuántos SAPS necesita el sistema, consulte con el integrador de sistemas o el asociado tecnológico de SAP.
    * **Disponibilidad del sistema**: Seleccione **Alta disponibilidad**.
    * **Nombre de usuario administrador, contraseña del administrador o clave SSH**: Se crea un usuario nuevo que se puede usar para iniciar sesión en la máquina.
    * **Identificador de subred**: Si quiere implementar la máquina virtual en una red virtual existente en la que tiene una subred definida a la que se debe asignar la máquina virtual, asigne un nombre al identificador de esa subred específica. Generalmente, el identificador tiene un aspecto similar al siguiente **/subscriptions/\<subscription ID>/resourceGroups/\<resource group name>/providers/Microsoft.Network/virtualNetworks/\<virtual network name>/subnets/\<subnet name>** . Deje el identificador en blanco si quiere crear una nueva red virtual

### <a name="manual-deployment"></a>Implementación manual

1. Cree un grupo de recursos.
1. Cree una red virtual.
1. Cree un conjunto de disponibilidad.  
   Establezca el dominio máximo de actualización.
1. Cree un equilibrador de carga (interno). Se recomienda que sea un [equilibrador de carga estándar](../../../load-balancer/load-balancer-overview.md).
   * Seleccione la red virtual que creó en el paso 2.
1. Cree la máquina virtual 1.  
   Use por lo menos Red Hat Enterprise Linux 7.4 para SAP HANA. Este ejemplo usa la imagen de Red Hat Enterprise Linux 7.4 para SAP HANA <https://portal.azure.com/#create/RedHat.RedHatEnterpriseLinux75forSAP-ARM> Seleccione el conjunto de disponibilidad creado en el paso 3.
1. Cree la máquina virtual 2.  
   Use por lo menos Red Hat Enterprise Linux 7.4 para SAP HANA. Este ejemplo usa la imagen de Red Hat Enterprise Linux 7.4 para SAP HANA <https://portal.azure.com/#create/RedHat.RedHatEnterpriseLinux75forSAP-ARM> Seleccione el conjunto de disponibilidad creado en el paso 3.
1. Agregue discos de datos.
1. Si usa Standard Load Balancer, siga estos pasos de configuración:
   1. Primero, cree un grupo de direcciones IP de front-end:

      1. Abra el equilibrador de carga, seleccione **frontend IP pool** (Grupo de direcciones IP de front-end) y haga clic en **Agregar**.
      1. Escriba el nombre del nuevo grupo de direcciones IP de front-end (por ejemplo, **hana-front-end**).
      1. Establezca **Asignación** en **Estática** y escriba la dirección IP (por ejemplo, **10.0.0.13**).
      1. Seleccione **Aceptar**.
      1. Una vez creado el nuevo grupo de direcciones IP de front-end, anote la dirección IP del grupo.

   1. A continuación, cree un grupo de back-end:

      1. Abra el equilibrador de carga, seleccione **Grupos de back-end** y haga clic en **Agregar**.
      1. Escriba el nombre del nuevo grupo de back-end (por ejemplo, **hana-backend**).
      1. Seleccione **Agregar una máquina virtual**.
      1. Seleccione **Máquina virtual**.
      1. Seleccione las máquinas virtuales del clúster de SAP HANA y sus direcciones IP.
      1. Seleccione **Agregar**.

   1. A continuación, cree un sondeo de estado:

      1. Abra el equilibrador de carga, seleccione **Sondeos de estado** y haga clic en **Agregar**.
      1. Escriba el nombre del sondeo de estado nuevo (por ejemplo **hana-hp**).
      1. Seleccione **TCP** como protocolo y el puerto 625**03**. Mantenga el valor de **Intervalo** en 5 y el valor de **Umbral incorrecto** en 2.
      1. Seleccione **Aceptar**.

   1. Luego cree las reglas de equilibrio de carga:
   
      1. Abra el equilibrador de carga, seleccione **Reglas de equilibrio de carga** y haga clic en **Agregar**.
      1. Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, **hana-lb**).
      1. Seleccione la dirección IP de front-end, el grupo de back-end y el sondeo de estado que ha creado anteriormente (por ejemplo, **hana-frontend**, **hana-backend** y **hana-hp**).
      1. Seleccione **Puertos HA**.
      1. Aumente el **tiempo de espera de inactividad** a 30 minutos.
      1. Asegúrese de **habilitar la dirección IP flotante**.
      1. Seleccione **Aceptar**.

   > [!Note]
   > Cuando las máquinas virtuales sin direcciones IP públicas se colocan en el grupo de back-end de Standard Load Balancer interno (sin dirección IP pública), no hay conectividad saliente de Internet, a menos que se realice una configuración adicional para permitir el enrutamiento a puntos de conexión públicos. Para obtener más información sobre cómo obtener conectividad saliente, vea [Conectividad de punto de conexión público para máquinas virtuales con Azure Standard Load Balancer en escenarios de alta disponibilidad de SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

1. Como alternativa, si el escenario dicta el uso de Basic Load Balancer, siga estos pasos de configuración:
   1. Configure el equilibrador de carga. Primero, cree un grupo de direcciones IP de front-end:

      1. Abra el equilibrador de carga, seleccione **frontend IP pool** (Grupo de direcciones IP de front-end) y haga clic en **Agregar**.
      1. Escriba el nombre del nuevo grupo de direcciones IP de front-end (por ejemplo, **hana-front-end**).
      1. Establezca **Asignación** en **Estática** y escriba la dirección IP (por ejemplo, **10.0.0.13**).
      1. Seleccione **Aceptar**.
      1. Una vez creado el nuevo grupo de direcciones IP de front-end, anote la dirección IP del grupo.

   1. A continuación, cree un grupo de back-end:

      1. Abra el equilibrador de carga, seleccione **Grupos de back-end** y haga clic en **Agregar**.
      1. Escriba el nombre del nuevo grupo de back-end (por ejemplo, **hana-backend**).
      1. Seleccione **Agregar una máquina virtual**.
      1. Seleccione el conjunto de disponibilidad creado en el paso 3.
      1. Seleccione las máquinas virtuales del clúster de SAP HANA
      1. Seleccione **Aceptar**.

   1. A continuación, cree un sondeo de estado:

      1. Abra el equilibrador de carga, seleccione **Sondeos de estado** y haga clic en **Agregar**.
      1. Escriba el nombre del sondeo de estado nuevo (por ejemplo **hana-hp**).
      1. Seleccione **TCP** como protocolo y el puerto 625**03**. Mantenga el valor de **Intervalo** en 5 y el valor de **Umbral incorrecto** en 2.
      1. Seleccione **Aceptar**.

   1. Para SAP HANA 1.0, cree las reglas de equilibrio de carga:

      1. Abra el equilibrador de carga, seleccione **Reglas de equilibrio de carga** y haga clic en **Agregar**.
      1. Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, hana-lb-3**03**15).
      1. Seleccione la dirección IP de front-end, el grupo de back-end y el sondeo de estado que creó anteriormente (por ejemplo, **hana-frontend**).
      1. Mantenga el valor de **Protocolo** en **TCP** y escriba el puerto 3**03**15.
      1. Aumente el **tiempo de espera de inactividad** a 30 minutos.
      1. Asegúrese de **habilitar la dirección IP flotante**.
      1. Seleccione **Aceptar**.
      1. Repita estos pasos para el puerto 3**03**17.

   1. Para SAP HANA 2.0, cree reglas de equilibrio de carga para la base de datos del sistema:

      1. Abra el equilibrador de carga, seleccione **Reglas de equilibrio de carga** y haga clic en **Agregar**.
      1. Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, hana-lb-3**03**13).
      1. Seleccione la dirección IP de front-end, el grupo de back-end y el sondeo de estado que creó anteriormente (por ejemplo, **hana-frontend**).
      1. Mantenga el valor de **Protocolo** en **TCP** y escriba el puerto 3**03**13.
      1. Aumente el **tiempo de espera de inactividad** a 30 minutos.
      1. Asegúrese de **habilitar la dirección IP flotante**.
      1. Seleccione **Aceptar**.
      1. Repita estos pasos para el puerto 3**03**14.

   1. Para SAP HANA 2.0, primero cree las reglas de equilibrio de carga para la base de datos de inquilino:

      1. Abra el equilibrador de carga, seleccione **Reglas de equilibrio de carga** y haga clic en **Agregar**.
      1. Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, hana-lb-3**03**40).
      1. Seleccione la dirección IP de front-end, el grupo de back-end y el sondeo de estado que creó anteriormente (por ejemplo, **hana-frontend**).
      1. Mantenga el valor de **Protocolo** en **TCP** y escriba el puerto 3**03**40.
      1. Aumente el **tiempo de espera de inactividad** a 30 minutos.
      1. Asegúrese de **habilitar la dirección IP flotante**.
      1. Seleccione **Aceptar**.
      1. Repita estos pasos para los puertos 3**03**41 y 3**03**42.

Para obtener más información sobre los puertos necesarios para SAP HANA, lea el capítulo [Connections to Tenant Databases](https://help.sap.com/viewer/78209c1d3a9b41cd8624338e42a12bf6/latest/en-US/7a9343c9f2a2436faa3cfdb5ca00c052.html) (Conexiones a las bases de datos de inquilino) de la guía [SAP HANA Tenant Databases](https://help.sap.com/viewer/78209c1d3a9b41cd8624338e42a12bf6) (Bases de datos de inquilino de SAP HANA) o la [nota de SAP 2388694][2388694].

> [!IMPORTANT]
> No habilite las marcas de tiempo TCP en VM de Azure que se encuentren detrás de Azure Load Balancer. Si habilita las marcas de tiempo TCP provocará un error en los sondeos de estado. Establezca el parámetro **net.ipv4.tcp_timestamps** a **0**. Consulte [Sondeos de estado de Load Balancer](../../../load-balancer/load-balancer-custom-probe-overview.md) para obtener más información.
> Consulte también la nota de SAP [2382421](https://launchpad.support.sap.com/#/notes/2382421). 

## <a name="install-sap-hana"></a>Instalación de SAP HANA

En los pasos de esta sección se usan los siguientes prefijos:

* **[A]** : el paso se aplica a todos los nodos.
* **[1]** : el paso solo se aplica al nodo 1.
* **[2]** : el paso solo se aplica al nodo 2 del clúster de Pacemaker.

1. **[A]** Configuración del diseño de disco: **Administrador de volúmenes lógicos (LVM)** .

   Se recomienda usar LVM para volúmenes que almacenen datos y archivos de registro. En el ejemplo siguiente se supone que las máquinas virtuales tienen cuatro discos de datos asociados que se usan para crear dos volúmenes.

   Enumere todos los discos disponibles:

   <pre><code>ls /dev/disk/azure/scsi1/lun*
   </code></pre>

   Salida de ejemplo:

   <pre><code>
   /dev/disk/azure/scsi1/lun0  /dev/disk/azure/scsi1/lun1  /dev/disk/azure/scsi1/lun2  /dev/disk/azure/scsi1/lun3
   </code></pre>

   Cree volúmenes físicos para todos los discos que quiera usar:

   <pre><code>sudo pvcreate /dev/disk/azure/scsi1/lun0
   sudo pvcreate /dev/disk/azure/scsi1/lun1
   sudo pvcreate /dev/disk/azure/scsi1/lun2
   sudo pvcreate /dev/disk/azure/scsi1/lun3
   </code></pre>

   Cree un grupo de volúmenes para los archivos de datos. Use un grupo de volúmenes para los archivos de registro y otro para el directorio compartido de SAP HANA:

   <pre><code>sudo vgcreate vg_hana_data_<b>HN1</b> /dev/disk/azure/scsi1/lun0 /dev/disk/azure/scsi1/lun1
   sudo vgcreate vg_hana_log_<b>HN1</b> /dev/disk/azure/scsi1/lun2
   sudo vgcreate vg_hana_shared_<b>HN1</b> /dev/disk/azure/scsi1/lun3
   </code></pre>

   Cree los volúmenes lógicos. Cuando se usa `lvcreate` sin el modificador `-i`, se crea un volumen lineal. Se recomienda crear un volumen seccionado para mejorar el rendimiento de E/S y, después, alinear los tamaños de sección con los valores documentados en [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md). El argumento `-i` debe ser el número de volúmenes físicos subyacentes y `-I`, el tamaño de la sección. En este documento, se usan dos volúmenes físicos para el volumen de datos, por lo que el modificador `-i` se establece en **2**. El tamaño de sección para el volumen de datos es **256 KiB**. Se usa un volumen físico para el volumen de registro, así que no se usa explícitamente ningún modificador `-i` o `-I` para los comandos del volumen de registro.  

   > [!IMPORTANT]
   > Use el modificador `-i` y establézcalo en el número del volumen físico subyacente cuando se usa más de un volumen físico para cada volumen de datos, de registro o compartido. Use el modificador `-I` para especificar el tamaño de las secciones al crear un volumen seccionado.  
   > Consulte [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md) para conocer las configuraciones de almacenamiento recomendadas, incluidos los tamaños de sección y el número de discos.  

   <pre><code>sudo lvcreate <b>-i 2</b> <b>-I 256</b> -l 100%FREE -n hana_data vg_hana_data_<b>HN1</b>
   sudo lvcreate -l 100%FREE -n hana_log vg_hana_log_<b>HN1</b>
   sudo lvcreate -l 100%FREE -n hana_shared vg_hana_shared_<b>HN1</b>
   sudo mkfs.xfs /dev/vg_hana_data_<b>HN1</b>/hana_data
   sudo mkfs.xfs /dev/vg_hana_log_<b>HN1</b>/hana_log
   sudo mkfs.xfs /dev/vg_hana_shared_<b>HN1</b>/hana_shared
   </code></pre>

   Cree los directorios de montaje y copie el UUID de todos los volúmenes lógicos:

   <pre><code>sudo mkdir -p /hana/data/<b>HN1</b>
   sudo mkdir -p /hana/log/<b>HN1</b>
   sudo mkdir -p /hana/shared/<b>HN1</b>
   # Write down the ID of /dev/vg_hana_data_<b>HN1</b>/hana_data, /dev/vg_hana_log_<b>HN1</b>/hana_log, and /dev/vg_hana_shared_<b>HN1</b>/hana_shared
   sudo blkid
   </code></pre>

   Cree entradas `fstab` para los tres volúmenes lógicos:

   <pre><code>sudo vi /etc/fstab
   </code></pre>

   Inserte la siguiente línea en el archivo `/etc/fstab`:

   <pre><code>/dev/disk/by-uuid/<b>&lt;UUID of /dev/mapper/vg_hana_data_<b>HN1</b>-hana_data&gt;</b> /hana/data/<b>HN1</b> xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<b>&lt;UUID of /dev/mapper/vg_hana_log_<b>HN1</b>-hana_log&gt;</b> /hana/log/<b>HN1</b> xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<b>&lt;UUID of /dev/mapper/vg_hana_shared_<b>HN1</b>-hana_shared&gt;</b> /hana/shared/<b>HN1</b> xfs  defaults,nofail  0  2
   </code></pre>

   Monte los nuevos volúmenes:

   <pre><code>sudo mount -a
   </code></pre>

1. **[A]** Configuración del diseño de disco: **Discos sin formato**.

   Para sistemas demostración, puede colocar los archivos de datos y de registro HANA en un disco. Cree una partición en /dev/disk/azure/scsi1/lun0 y aplíquele formato con xfs:

   <pre><code>sudo sh -c 'echo -e "n\n\n\n\n\nw\n" | fdisk /dev/disk/azure/scsi1/lun0'
   sudo mkfs.xfs /dev/disk/azure/scsi1/lun0-part1
   
   # Write down the ID of /dev/disk/azure/scsi1/lun0-part1
   sudo /sbin/blkid
   sudo vi /etc/fstab
   </code></pre>

   Inserte esta línea en el archivo/etc/fstab:

   <pre><code>/dev/disk/by-uuid/<b>&lt;UUID&gt;</b> /hana xfs  defaults,nofail  0  2
   </code></pre>

   Cree el directorio de destino y monte el disco:

   <pre><code>sudo mkdir /hana
   sudo mount -a
   </code></pre>

1. **[A]** Configuración de la resolución del nombre de todos los hosts.

   Puede usar un servidor DNS o modificar el archivo /etc/hosts en todos los nodos. En este ejemplo se muestra cómo usar el archivo /etc/hosts.
   Reemplace la dirección IP y el nombre de host en los siguientes comandos:

   <pre><code>sudo vi /etc/hosts
   </code></pre>

   Inserte las líneas siguientes en el archivo/etc/hosts. Cambie la dirección IP y el nombre de host para que coincidan con su entorno:

   <pre><code><b>10.0.0.5 hn1-db-0</b>
   <b>10.0.0.6 hn1-db-1</b>
   </code></pre>

1. **[A]** Configuración de RHEL para HANA

   Configure RHEL tal como se describe en las notas de SAP [2292690], [2455582] y <https://access.redhat.com/solutions/2447641>.

1. **[A]** Instalación de SAP HANA

   Para instalar la replicación del sistema de SAP HANA, siga los pasos en <https://access.redhat.com/articles/3004101>.

   * Ejecute el programa **hdblcm** del DVD de HANA. Escriba los siguientes valores en el símbolo del sistema:
   * Elija la instalación: Especifique **1**.
   * Seleccione los componentes adicionales para la instalación: Especifique **1**.
   * Escriba la ruta de acceso de instalación [/hana/shared]: Presione Entrar.
   * Enter Local Host Name [..]: Presione Entrar.
   * ¿Desea agregar hosts adicionales al sistema? (s/n) [n]: Presione Entrar.
   * Escriba el identificador del sistema de SAP HANA: Escriba el SID de HANA, por ejemplo: **HN1**.
   * Escriba el número de instancia [00]: Escriba el número de instancia de HANA. Escribir **03** si usó la plantilla de Azure o ha seguido la sección de implementación manual de este artículo.
   * Seleccione Database Mode (Modo de base de datos) / escriba el índice [1]: Presione Entrar.
   * Seleccione Uso del sistema / escriba el índice [4]: Seleccione el valor de uso del sistema.
   * Escriba la ubicación de los volúmenes de datos [/hana/data/HN1]: Presione Entrar.
   * Escriba la ubicación de los volúmenes de registros [/hana/log/HN1]: Presione Entrar.
   * ¿Restringir la asignación de memoria máxima? [n]: Presione Entrar.
   * Escriba el nombre del host del certificado para el host '...' [...]: Presione Entrar.
   * Escriba la contraseña del usuario del agente de host de SAP (sapadm): Escriba la contraseña de usuario del agente de host.
   * Confirme la contraseña del usuario del agente de host de SAP (sapadm): Escriba de nuevo la contraseña de usuario del agente de host para confirmarla.
   * Escriba la contraseña del administrador del sistema (hdbadm): Escriba la contraseña de administrador del sistema.
   * Confirme la contraseña del administrador del sistema (hdbadm): Escriba de nuevo la contraseña de administrador del sistema para confirmarla.
   * Escriba el directorio principal del administrador de sistema [/usr/sap/HN1/home]: Presione Entrar.
   * Escriba el shell de inicio de sesión del administrador de sistema [/bin/sh]: Presione Entrar.
   * Escriba el identificador de usuario del administrador de sistema [1001]: Presione Entrar.
   * Escriba el identificador del grupo de usuarios (sapsys) [79]: Presione Entrar.
   * Escriba la contraseña del usuario (SYSTEM) de la base de datos: Escriba la contraseña de usuario de base de datos.
   * Confirme la contraseña del usuario (SYSTEM) de la base de datos: Escriba de nuevo la contraseña de usuario de base de datos para confirmarla.
   * ¿Reiniciar el sistema tras el reinicio de la máquina? [n]: Presione Entrar.
   * ¿Desea continuar? (s/n): valide el resumen. Escriba **s** para continuar.

1. **[A]** Actualización del agente de host de SAP.

   Descargue el archivo más reciente del agente de host de SAP desde [SAP Software Center][sap-swcenter] y ejecute el siguiente comando para actualizar el agente. Reemplace la ruta de acceso al archivo para que apunte al archivo que descargó:

   <pre><code>sudo /usr/sap/hostctrl/exe/saphostexec -upgrade -archive &lt;path to SAP Host Agent SAR&gt;
   </code></pre>

1. **[A]** Configuración del firewall

   Cree la regla de firewall para el puerto de sondeo del equilibrador de carga de Azure.

   <pre><code>sudo firewall-cmd --zone=public --add-port=625<b>03</b>/tcp
   sudo firewall-cmd --zone=public --add-port=625<b>03</b>/tcp --permanent
   </code></pre>

## <a name="configure-sap-hana-20-system-replication"></a>Configuración de la Replicación del sistema de SAP HANA 2.0

En los pasos de esta sección se usan los siguientes prefijos:

* **[A]** : el paso se aplica a todos los nodos.
* **[1]** : el paso solo se aplica al nodo 1.
* **[2]** : el paso solo se aplica al nodo 2 del clúster de Pacemaker.

1. **[A]** Configuración del firewall

   Cree reglas de firewall para permitir la replicación del sistema de HANA y el tráfico del cliente. Los puertos necesarios se enumeran en [TCP/IP Ports of All SAP Products](https://help.sap.com/viewer/ports) (Puertos TCP/IP de todos los productos de SAP). Los siguientes comandos son simplemente un ejemplo para permitir la replicación del sistema de HANA 2.0 y el tráfico del cliente a las bases de datos SYSTEMDB, HN1 y NW1.

   <pre><code>sudo firewall-cmd --zone=public --add-port=40302/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40302/tcp
   sudo firewall-cmd --zone=public --add-port=40301/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40301/tcp
   sudo firewall-cmd --zone=public --add-port=40307/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40307/tcp
   sudo firewall-cmd --zone=public --add-port=40303/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40303/tcp
   sudo firewall-cmd --zone=public --add-port=40340/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40340/tcp
   sudo firewall-cmd --zone=public --add-port=30340/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=30340/tcp
   sudo firewall-cmd --zone=public --add-port=30341/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=30341/tcp
   sudo firewall-cmd --zone=public --add-port=30342/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=30342/tcp
   </code></pre>

1. **[1]** Creación de la base de datos de inquilino.

   Si usa SAP HANA 2.0 o MDC, cree una base de datos de inquilino para el sistema SAP NetWeaver. Reemplace **NW1** por el SID del sistema SAP.

   Ejecute el siguiente comando como < hanasid\>adm:

   <pre><code>hdbsql -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> -d SYSTEMDB 'CREATE DATABASE <b>NW1</b> SYSTEM USER PASSWORD "<b>passwd</b>"'
   </code></pre>

1. **[1]** Configuración de la replicación del sistema en el primer nodo:

   Realice una copia de seguridad de las bases de datos como < hanasid\>adm:

   <pre><code>hdbsql -d SYSTEMDB -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackupSYS</b>')"
   hdbsql -d <b>HN1</b> -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackupHN1</b>')"
   hdbsql -d <b>NW1</b> -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackupNW1</b>')"
   </code></pre>

   Copie los archivos PKI de sistema en el sitio secundario:

   <pre><code>scp /usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/data/SSFS_<b>HN1</b>.DAT   <b>hn1-db-1</b>:/usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/data/
   scp /usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/key/SSFS_<b>HN1</b>.KEY  <b>hn1-db-1</b>:/usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/key/
   </code></pre>

   Cree el sitio principal:

   <pre><code>hdbnsutil -sr_enable --name=<b>SITE1</b>
   </code></pre>

1. **[2]** Configuración de la replicación del sistema en el segundo nodo:
    
   Registre el segundo nodo para iniciar la replicación del sistema. Ejecute el siguiente comando como <hanasid\>adm:

   <pre><code>sapcontrol -nr <b>03</b> -function StopWait 600 10
   hdbnsutil -sr_register --remoteHost=<b>hn1-db-0</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE2</b>
   </code></pre>

1. **[1]** Comprobación del estado de replicación

   Compruebe el estado de replicación y espere hasta que todas las bases de datos estén sincronizadas. Si el estado sigue siendo DESCONOCIDO, compruebe la configuración del firewall.

   <pre><code>sudo su - <b>hn1</b>adm -c "python /usr/sap/<b>HN1</b>/HDB<b>03</b>/exe/python_support/systemReplicationStatus.py"
   # | Database | Host     | Port  | Service Name | Volume ID | Site ID | Site Name | Secondary | Secondary | Secondary | Secondary | Secondary     | Replication | Replication | Replication    |
   # |          |          |       |              |           |         |           | Host      | Port      | Site ID   | Site Name | Active Status | Mode        | Status      | Status Details |
   # | -------- | -------- | ----- | ------------ | --------- | ------- | --------- | --------- | --------- | --------- | --------- | ------------- | ----------- | ----------- | -------------- |
   # | SYSTEMDB | <b>hn1-db-0</b> | 30301 | nameserver   |         1 |       1 | <b>SITE1</b>     | <b>hn1-db-1</b>  |     30301 |         2 | <b>SITE2</b>     | YES           | SYNC        | ACTIVE      |                |
   # | <b>HN1</b>      | <b>hn1-db-0</b> | 30307 | xsengine     |         2 |       1 | <b>SITE1</b>     | <b>hn1-db-1</b>  |     30307 |         2 | <b>SITE2</b>     | YES           | SYNC        | ACTIVE      |                |
   # | <b>NW1</b>      | <b>hn1-db-0</b> | 30340 | indexserver  |         2 |       1 | <b>SITE1</b>     | <b>hn1-db-1</b>  |     30340 |         2 | <b>SITE2</b>     | YES           | SYNC        | ACTIVE      |                |
   # | <b>HN1</b>      | <b>hn1-db-0</b> | 30303 | indexserver  |         3 |       1 | <b>SITE1</b>     | <b>hn1-db-1</b>  |     30303 |         2 | <b>SITE2</b>     | YES           | SYNC        | ACTIVE      |                |
   #
   # status system replication site "2": ACTIVE
   # overall system replication status: ACTIVE
   #
   # Local System Replication State
   # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   #
   # mode: PRIMARY
   # site id: 1
   # site name: <b>SITE1</b>
   </code></pre>

## <a name="configure-sap-hana-10-system-replication"></a>Configuración de la Replicación del sistema de SAP HANA 1.0

En los pasos de esta sección se usan los siguientes prefijos:

* **[A]** : el paso se aplica a todos los nodos.
* **[1]** : el paso solo se aplica al nodo 1.
* **[2]** : el paso solo se aplica al nodo 2 del clúster de Pacemaker.

1. **[A]** Configuración del firewall

   Cree reglas de firewall para permitir la replicación del sistema de HANA y el tráfico del cliente. Los puertos necesarios se enumeran en [TCP/IP Ports of All SAP Products](https://help.sap.com/viewer/ports) (Puertos TCP/IP de todos los productos de SAP). Los siguientes comandos son simplemente un ejemplo para permitir la replicación del sistema de HANA 2.0. Adáptelo a su instalación de SAP HANA 1.0.

   <pre><code>sudo firewall-cmd --zone=public --add-port=40302/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40302/tcp
   </code></pre>

1. **[1]** Creación de los usuarios necesarios.

   Ejecute el siguiente comando como raíz. Asegúrese de reemplazar las cadenas en negrita (**HN1** del identificador de sistema de HANA y el número de instancia **03**) por los valores de la instalación de SAP HANA:

   <pre><code>PATH="$PATH:/usr/sap/<b>HN1</b>/HDB<b>03</b>/exe"
   hdbsql -u system -i <b>03</b> 'CREATE USER <b>hdb</b>hasync PASSWORD "<b>passwd</b>"'
   hdbsql -u system -i <b>03</b> 'GRANT DATA ADMIN TO <b>hdb</b>hasync'
   hdbsql -u system -i <b>03</b> 'ALTER USER <b>hdb</b>hasync DISABLE PASSWORD LIFETIME'
   </code></pre>

1. **[A]** Creación de la entrada del almacén de claves.

   Ejecute el siguiente comando como raíz para crear una entrada del almacén de claves:

   <pre><code>PATH="$PATH:/usr/sap/<b>HN1</b>/HDB<b>03</b>/exe"
   hdbuserstore SET <b>hdb</b>haloc localhost:3<b>03</b>15 <b>hdb</b>hasync <b>passwd</b>
   </code></pre>

1. **[1]** Realización de una copia de seguridad de la base de datos.

   Haga una copia de seguridad de las bases de datos como raíz:

   <pre><code>PATH="$PATH:/usr/sap/<b>HN1</b>/HDB<b>03</b>/exe"
   hdbsql -d SYSTEMDB -u system -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackup</b>')"
   </code></pre>

   Si usa una instalación multiinquilino, haga también una copia de seguridad de la base de datos de inquilino:

   <pre><code>hdbsql -d <b>HN1</b> -u system -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackup</b>')"
   </code></pre>

1. **[1]** Configuración de la replicación del sistema en el primer nodo.

   Cree el sitio principal como <hanasid\>adm:

   <pre><code>su - <b>hdb</b>adm
   hdbnsutil -sr_enable –-name=<b>SITE1</b>
   </code></pre>

1. **[2]** Configuración de la replicación del sistema en el nodo secundario.

   Registre el sitio secundario como <hanasid\>adm:

   <pre><code>HDB stop
   hdbnsutil -sr_register --remoteHost=<b>hn1-db-0</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE2</b>
   HDB start
   </code></pre>

## <a name="create-a-pacemaker-cluster"></a>Creación de un clúster de Pacemaker

Siga los pasos de [Configuración de Pacemaker en Red Hat Enterprise Linux en Azure](high-availability-guide-rhel-pacemaker.md) para crear un clúster de Pacemaker básico para este servidor HANA.

## <a name="create-sap-hana-cluster-resources"></a>Creación de recursos de clúster de SAP HANA

Instale los agentes de recursos de SAP HANA en **todos los nodos**. Asegúrese de habilitar un repositorio que contenga el paquete.

<pre><code># Enable repository that contains SAP HANA resource agents
sudo subscription-manager repos --enable="rhel-sap-hana-for-rhel-7-server-rpms"
   
sudo yum install -y resource-agents-sap-hana
</code></pre>

A continuación, cree la topología de HANA. Ejecute los comandos siguientes en uno de los nodos del clúster de Pacemaker:

<pre><code>sudo pcs property set maintenance-mode=true

# Replace the bold string with your instance number and HANA system ID
sudo pcs resource create SAPHanaTopology_<b>HN1</b>_<b>03</b> SAPHanaTopology SID=<b>HN1</b> InstanceNumber=<b>03</b> \
op start timeout=600 op stop timeout=300 op monitor interval=10 timeout=600 \
--clone clone-max=2 clone-node-max=1 interleave=true
</code></pre>

A continuación, cree los recursos de HANA:

<pre><code># Replace the bold string with your instance number, HANA system ID, and the front-end IP address of the Azure load balancer.

sudo pcs resource create SAPHana_<b>HN1</b>_<b>03</b> SAPHana SID=<b>HN1</b> InstanceNumber=<b>03</b> PREFER_SITE_TAKEOVER=true DUPLICATE_PRIMARY_TIMEOUT=7200 AUTOMATED_REGISTER=false \
op start timeout=3600 op stop timeout=3600 \
op monitor interval=61 role="Slave" timeout=700 \
op monitor interval=59 role="Master" timeout=700 \
op promote timeout=3600 op demote timeout=3600 \
master notify=true clone-max=2 clone-node-max=1 interleave=true

sudo pcs resource create vip_<b>HN1</b>_<b>03</b> IPaddr2 ip="<b>10.0.0.13</b>"

sudo pcs resource create nc_<b>HN1</b>_<b>03</b> azure-lb port=625<b>03</b>

sudo pcs resource group add g_ip_<b>HN1</b>_<b>03</b> nc_<b>HN1</b>_<b>03</b> vip_<b>HN1</b>_<b>03</b>

sudo pcs constraint order SAPHanaTopology_<b>HN1</b>_<b>03</b>-clone then SAPHana_<b>HN1</b>_<b>03</b>-master symmetrical=false

sudo pcs constraint colocation add g_ip_<b>HN1</b>_<b>03</b> with master SAPHana_<b>HN1</b>_<b>03</b>-master 4000

sudo pcs property set maintenance-mode=false
</code></pre>

Asegúrese de que el estado del clúster sea el correcto y de que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.

> [!NOTE]
> Los tiempos de espera de la configuración anterior son solo ejemplos y puede ser necesario adaptarlos a la configuración específica de HANA. Por ejemplo, puede que necesite aumentar el tiempo de espera de inicio si la base de datos de SAP HANA tarda más en iniciarse.  

<pre><code>sudo pcs status

# Online: [ hn1-db-0 hn1-db-1 ]
#
# Full list of resources:
#
# azure_fence     (stonith:fence_azure_arm):      Started hn1-db-0
#  Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
#      Started: [ hn1-db-0 hn1-db-1 ]
#  Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
#      Masters: [ hn1-db-0 ]
#      Slaves: [ hn1-db-1 ]
#  Resource Group: g_ip_HN1_03
#      nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
#      vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>

## <a name="test-the-cluster-setup"></a>Prueba de la configuración del clúster

En esta sección se describe cómo se puede probar la configuración. Antes de comenzar la prueba, asegúrese de que Pacemaker no tenga acciones con error (mediante pcs status), no haya restricciones de ubicación inesperadas (por ejemplo, restos de una prueba de migración) y que HANA se encuentre en estado de sincronización, por ejemplo con systemReplicationStatus:

<pre><code>[root@hn1-db-0 ~]# sudo su - hn1adm -c "python /usr/sap/HN1/HDB03/exe/python_support/systemReplicationStatus.py"
</code></pre>

### <a name="test-the-migration"></a>Prueba de la migración

Estado del recurso antes de iniciar la prueba:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-0 ]
    Slaves: [ hn1-db-1 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>

Puede migrar el nodo maestro de SAP HANA con el siguiente comando:

<pre><code>[root@hn1-db-0 ~]# pcs resource move SAPHana_HN1_03-master
</code></pre>

Si estableció `AUTOMATED_REGISTER="false"`, este comando migrará el nodo maestro de SAP HANA y el grupo que contiene la dirección IP virtual a hn1-db-1.

Una vez realizada la migración, la salida de "sudo pcs status" se parece a esta

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-1 ]
    Stopped: [ hn1-db-0 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-1
</code></pre>

Se ha detenido el recurso de SAP HANA en hn1-db-0. En este caso, configure la instancia de HANA como secundaria mediante la ejecución de este comando:

<pre><code>[root@hn1-db-0 ~]# su - hn1adm

# Stop the HANA instance just in case it is running
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> sapcontrol -nr 03 -function StopWait 600 10
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-1 --remoteInstance=03 --replicationMod
e=sync --name=SITE1
</code></pre>

La migración crea restricciones de ubicación que deben eliminarse de nuevo:

<pre><code># Switch back to root
exit
[root@hn1-db-0 ~]# pcs resource clear SAPHana_HN1_03-master
</code></pre>

Supervise el estado del recurso de HANA mediante "pcs status". Una vez iniciado HANA en hn1-db-0, la salida se parecerá a esta:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-1 ]
    Slaves: [ hn1-db-0 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-1
</code></pre>

### <a name="test-the-azure-fencing-agent"></a>Prueba del agente de delimitación de Azure

Estado del recurso antes de iniciar la prueba:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-1 ]
    Slaves: [ hn1-db-0 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-1
</code></pre>

Puede probar la configuración del agente de delimitación de Azure si deshabilita la interfaz de red en el nodo en el que SAP HANA está ejecutándose como maestro.
Consulte en el [artículo de Knowledgebase de Red Hat 79523](https://access.redhat.com/solutions/79523) la descripción de cómo simular un error de red. En este ejemplo se usa el script net_breaker para bloquear todo el acceso a la red.

<pre><code>[root@hn1-db-1 ~]# sh ./net_breaker.sh BreakCommCmd 10.0.0.6
</code></pre>

Ahora, la máquina virtual se reiniciará o se detendrá, según la configuración del clúster.
Si estableció el valor `stonith-action` como desactivado, la máquina virtual se detendrá y los recursos se migrarán a la máquina virtual en ejecución.

Después de volver a iniciar la máquina virtual, el recurso de SAP HANA no se podrá iniciar como secundario si estableció `AUTOMATED_REGISTER="false"`. En este caso, configure la instancia de HANA como secundaria mediante la ejecución de este comando:

<pre><code>su - <b>hn1</b>adm

# Stop the HANA instance just in case it is running
hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> sapcontrol -nr <b>03</b> -function StopWait 600 10
hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=<b>hn1-db-0</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE2</b>

# Switch back to root and clean up the failed state
exit
[root@hn1-db-1 ~]# pcs resource cleanup SAPHana_HN1_03-master
</code></pre>

Estado del recurso después de la prueba:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-0 ]
    Slaves: [ hn1-db-1 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>

### <a name="test-a-manual-failover"></a>Prueba de una conmutación por error manual

Estado del recurso antes de iniciar la prueba:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-0 ]
    Slaves: [ hn1-db-1 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>

Para probar una conmutación por error manual, detenga el clúster en el nodo hn1-db-0:

<pre><code>[root@hn1-db-0 ~]# pcs cluster stop
</code></pre>

Después de la conmutación por error, puede reiniciar el clúster. Si establece `AUTOMATED_REGISTER="false"`, el recurso de SAP HANA en el nodo hn1-db-0 no se podrá iniciar como secundario. En este caso, configure la instancia de HANA como secundaria mediante la ejecución de este comando:

<pre><code>[root@hn1-db-0 ~]# pcs cluster start
[root@hn1-db-0 ~]# su - hn1adm

# Stop the HANA instance just in case it is running
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> sapcontrol -nr 03 -function StopWait 600 10
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=<b>hn1-db-1</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE1</b>

# Switch back to root and clean up the failed state
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> exit
[root@hn1-db-1 ~]# pcs resource cleanup SAPHana_HN1_03-master
</code></pre>

Estado del recurso después de la prueba:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-1 ]
     Slaves: [ hn1-db-0 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-1
</code></pre>

### <a name="test-a-manual-failover"></a>Prueba de una conmutación por error manual

Estado del recurso antes de iniciar la prueba:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-0 ]
    Slaves: [ hn1-db-1 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>

Para probar una conmutación por error manual, detenga el clúster en el nodo hn1-db-0:

<pre><code>[root@hn1-db-0 ~]# pcs cluster stop
</code></pre>


## <a name="next-steps"></a>Pasos siguientes

* [Planeamiento e implementación de Azure Virtual Machines para SAP][planning-guide]
* [Implementación de Azure Virtual Machines para SAP][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP][dbms-guide]
* [Configuraciones de almacenamiento de máquina virtual SAP HANA](./hana-vm-operations-storage.md)
