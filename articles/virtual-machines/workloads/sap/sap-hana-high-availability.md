---
title: Alta disponibilidad de SAP HANA en máquinas virtuales de Azure en SLES | Microsoft Docs
description: Alta disponibilidad de SAP HANA en máquinas virtuales de Azure en SUSE Linux Enterprise Server
services: virtual-machines-linux
documentationcenter: ''
author: rdeltcheva
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/11/2020
ms.author: radeltch
ms.openlocfilehash: f2b4b207aca92cc37b71f3cb12ec579a6b57e832
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87068955"
---
# <a name="high-availability-of-sap-hana-on-azure-vms-on-suse-linux-enterprise-server"></a>Alta disponibilidad de SAP HANA en máquinas virtuales de Azure en SUSE Linux Enterprise Server

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
[401162]:https://launchpad.support.sap.com/#/notes/401162

[hana-ha-guide-replication]:sap-hana-high-availability.md#14c19f65-b5aa-4856-9594-b81c7e4df73d
[hana-ha-guide-shared-storage]:sap-hana-high-availability.md#498de331-fa04-490b-997c-b078de457c9d
[sles-for-sap-bp]:https://www.suse.com/documentation/sles-for-sap-12/

[suse-hana-ha-guide]:https://www.suse.com/docrep/documents/ir8w88iwu7/suse_linux_enterprise_server_for_sap_applications_12_sp1.pdf
[sap-swcenter]:https://launchpad.support.sap.com/#/softwarecenter
[template-multisid-db]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[template-converged]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-converged-md%2Fazuredeploy.json

Para el desarrollo local, puede usar la replicación del sistema de HANA o el almacenamiento compartido para establecer alta disponibilidad para SAP HANA.
En las máquinas virtuales (VM) de Azure, la replicación del sistema de HANA en Azure es actualmente la única función admitida de alta disponibilidad. La replicación de SAP HANA se realiza con un nodo principal y al menos uno secundario. Los cambios en los datos del nodo principal se replican en los secundarios de forma sincrónica o asincrónica.

En este artículo se describe cómo implementar y configurar las máquinas virtuales, instalar la plataforma del clúster e instalar y configurar la replicación del sistema de SAP HANA.
En las configuraciones de ejemplo, se usan los comandos de instalación, el número de instancia **03** y el identificador del sistema de HANA **HN1**.

Lea primero las notas y los documentos de SAP siguientes:

* Nota de SAP [1928533], que incluye:
  * La lista de tamaños de máquina virtual de Azure que se admiten para la implementación del software de SAP.
  * Información importante sobre capacidad para los tamaños de máquina virtual de Azure.
  * El software de SAP admitido y combinaciones de sistema operativo y base de datos.
  * La versión del kernel de SAP necesaria para Windows y Linux en Microsoft Azure.
* La nota de SAP [2015553] enumera los requisitos previos para las implementaciones de software de SAP admitidas por SAP en Azure.
* La nota de SAP [2205917] contiene configuraciones recomendadas del sistema operativo para SUSE Linux Enterprise Server para SAP Applications.
* La nota de SAP [1944799] contiene guías de SAP HANA para SUSE Linux Enterprise Server para SAP Applications.
* La nota de SAP [2178632] contiene información detallada sobre todas las métricas de supervisión notificadas para SAP en Azure.
* La nota de SAP [2191498] incluye la versión de SAP Host Agent necesaria para Linux en Azure.
* La nota de SAP [2243692] incluye información acerca de las licencias de SAP en Linux en Azure.
* La nota de SAP [1984787] incluye información general sobre SUSE Linux Enterprise Server 12.
* La nota de SAP [1999351] contiene más soluciones de problemas de la extensión de supervisión mejorada de Azure para SAP.
* La nota de SAP [401162] contiene información sobre cómo evitar un mensaje semejante a "la dirección ya está en uso" al configurar la replicación del sistema de HANA.
* La [WIKI de la comunidad SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contiene todas las notas de SAP necesarias para Linux.
* [Plataformas IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure)
* Guía [Implementación y planeamiento de Azure Virtual Machines para SAP en Linux][planning-guide].
* [Implementación de Azure Virtual Machines para SAP en Linux][deployment-guide] (este artículo).
* Guía [Implementación de DBMS de Azure Virtual Machines para SAP en Linux][dbms-guide].
* [Guías de procedimientos recomendados de SUSE Linux Enterprise Server for SAP Applications 12 SP3][sles-for-sap-bp]
  * Configuración de una infraestructura optimizada para el rendimiento de SR de SAP HANA (SLES para SAP Applications 12 SP1). La guía contiene toda la información necesaria para configurar la replicación del sistema de SAP HANA para el desarrollo en el entorno local. Esta guía sirve como orientación.
  * Configuración de una infraestructura optimizada para los costos de SR de SAP HANA (SLES para SAP Applications 12 SP1).

## <a name="overview"></a>Información general

Para lograr una alta disponibilidad, SAP HANA se instala en dos máquinas virtuales. Los datos se replican mediante la replicación del sistema de HANA.

![Información general sobre alta disponibilidad de SAP HANA](./media/sap-hana-high-availability/ha-suse-hana.png)

En la instalación de la replicación del sistema de SAP HANA se usa un nombre de host virtual dedicado y direcciones IP virtuales. En Azure, se requiere un equilibrador de carga para usar una dirección IP virtual. En la lista siguiente se muestra la configuración del equilibrador de carga:

* Configuración de front-end: Dirección IP 10.0.0.13 para hn1-db
* Configuración de back-end: Se conecta a interfaces de red principales de todas las máquinas virtuales que deben formar parte de la Replicación del sistema de HANA.
* Puerto de sondeo: Puerto 62503
* Reglas de equilibrio de carga: 30313 TCP, 30315 TCP, 30317 TCP

## <a name="deploy-for-linux"></a>Implementación para Linux

El agente de recursos para SAP HANA se incluye en SUSE Linux Enterprise Server para SAP Applications.
Azure Marketplace contiene una imagen de SUSE Linux Enterprise Server para SAP Applications 12 que puede usar para implementar nuevas máquinas virtuales.

### <a name="deploy-with-a-template"></a>Implementación con una plantilla

Para implementar todos los recursos necesarios, puede usar una de las plantillas de inicio rápido de GitHub. La plantilla implementa las máquinas virtuales, el equilibrador de carga, el conjunto de disponibilidad, etc.
Para implementar la plantilla, siga estos pasos:

1. Abra la [plantilla de base de datos][template-multisid-db] o la [plantilla convergente][template-converged] en Azure Portal. 
    La plantilla de base de datos crea las reglas de equilibrio de carga solo para una base de datos. La plantilla convergida también crea las reglas de equilibrio de carga para una instancia de ASCS/SCS y ERS (solo Linux). Si tiene previsto instalar un sistema basado en SAP NetWeaver y desea instalar la instancia de ASCS/SCS en las mismas máquinas, use la [plantilla convergente][template-converged].

1. Escriba los siguientes parámetros:
    - **Identificador de sistema SAP**: Escriba el identificador del sistema SAP que se va a instalar. El identificador se usa como prefijo de los recursos que se implementan.
    - **Tipo de pila**: (Este parámetro solo es aplicable si usa la plantilla convergida). Seleccione el tipo de pila de SAP NetWeaver.
    - **Tipo de SO**: Seleccione una de las distribuciones de Linux. En este ejemplo, seleccione **SLES 12**.
    - **Tipo de base de datos**: Seleccione **HANA**.
    - **Tamaño del sistema SAP**: Escriba la cantidad de SAPS que va a proporcionar el nuevo sistema. Si no está seguro de cuántos SAPS necesita el sistema, consulte con el integrador de sistemas o el asociado tecnológico de SAP.
    - **Disponibilidad del sistema**: Seleccione **Alta disponibilidad**.
    - **Nombre de usuario y contraseña del administrador**: Se crea un usuario nuevo que se puede usar para iniciar sesión en la máquina.
    - **Subred nueva o existente**: Determina si es necesario crear una red virtual y subred nuevas o usar una subred existente. Si ya tiene una red virtual conectada a la red local, seleccione **Existing** (Existente).
    - **Identificador de subred**: Si quiere implementar la máquina virtual en una red virtual existente en la que tiene una subred definida a la que se debe asignar la máquina virtual, asigne un nombre al identificador de esa subred específica. Normalmente, el identificador se parece a **/subscriptions/\<subscription ID>/resourceGroups/\<resource group name>/providers/Microsoft.Network/virtualNetworks/\<virtual network name>/subnets/\<subnet name>** .

### <a name="manual-deployment"></a>Implementación manual

> [!IMPORTANT]
> Asegúrese de que el sistema operativo seleccionado está certificado por SAP para SAP HANA en los tipos específicos de máquina virtual que use. La lista de tipos de máquina virtual certificados para SAP HANA y las versiones de sistema operativo correspondientes se puede consultar en [Plataformas de IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). No olvide consultar la información del tipo de máquina virtual mostrado para obtener una lista completa de las versiones de sistema operativo que SAP HANA admite para ese tipo específico de máquina virtual.
>  

1. Cree un grupo de recursos.
1. Cree una red virtual.
1. Cree un conjunto de disponibilidad.
   - Establezca el dominio máximo de actualización.
1. Cree un equilibrador de carga (interno). Se recomienda que sea un [equilibrador de carga estándar](../../../load-balancer/load-balancer-overview.md).
   - Seleccione la red virtual que creó en el paso 2.
1. Cree la máquina virtual 1.
   - Use una imagen SLES4SAP en la galería de Azure que sea compatible con SAP HANA en el tipo de máquina virtual seleccionado.
   - Seleccione el conjunto de disponibilidad creado en el paso 3.
1. Cree la máquina virtual 2.
   - Use una imagen SLES4SAP en la galería de Azure que sea compatible con SAP HANA en el tipo de máquina virtual seleccionado.
   - Seleccione el conjunto de disponibilidad creado en el paso 3. 
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
      1. Seleccione **Virtual Network**.
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

## <a name="create-a-pacemaker-cluster"></a>Creación de un clúster de Pacemaker

Siga los pasos de [Configuración de Pacemaker en SUSE Linux Enterprise Server en Azure](high-availability-guide-suse-pacemaker.md) para crear un clúster de Pacemaker básico para este servidor HANA. También puede usar el mismo clúster de Pacemaker con SAP HANA y SAP NetWeaver (A)SCS.

## <a name="install-sap-hana"></a>Instalación de SAP HANA

En los pasos de esta sección se usan los siguientes prefijos:
- **[A]** : el paso se aplica a todos los nodos.
- **[1]** : el paso solo se aplica al nodo 1.
- **[2]** : el paso solo se aplica al nodo 2 del clúster de Pacemaker.

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

1. **[A]** Instalación de los paquetes de alta disponibilidad de SAP HANA:

   <pre><code>sudo zypper install SAPHanaSR
   </code></pre>

Para instalar la replicación del sistema de SAP HANA, siga el capítulo 4 de la [guía del escenario optimizado para el rendimiento de SR de SAP HANA](https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/).

1. **[A]** Ejecución del programa **hdblcm** desde el DVD de HANA. Escriba los siguientes valores en el símbolo del sistema:
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

## <a name="configure-sap-hana-20-system-replication"></a>Configuración de la Replicación del sistema de SAP HANA 2.0

En los pasos de esta sección se usan los siguientes prefijos:

* **[A]** : el paso se aplica a todos los nodos.
* **[1]** : el paso solo se aplica al nodo 1.
* **[2]** : el paso solo se aplica al nodo 2 del clúster de Pacemaker.

1. **[1]** Creación de la base de datos de inquilino.

   Si usa SAP HANA 2.0 o MDC, cree una base de datos de inquilino para el sistema SAP NetWeaver. Reemplace **NW1** por el SID del sistema SAP.

   Ejecute el siguiente comando como < hanasid\>adm:

   <pre><code>hdbsql -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> -d SYSTEMDB 'CREATE DATABASE <b>NW1</b> SYSTEM USER PASSWORD "<b>passwd</b>"'
   </code></pre>

1. **[1]** Configuración de la replicación del sistema en el primer nodo:

   Haga una copia de seguridad de las bases de datos como <hanasid\>adm:

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

## <a name="configure-sap-hana-10-system-replication"></a>Configuración de la Replicación del sistema de SAP HANA 1.0

En los pasos de esta sección se usan los siguientes prefijos:

* **[A]** : el paso se aplica a todos los nodos.
* **[1]** : el paso solo se aplica al nodo 1.
* **[2]** : el paso solo se aplica al nodo 2 del clúster de Pacemaker.

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

   <pre><code>sapcontrol -nr <b>03</b> -function StopWait 600 10
   hdbnsutil -sr_register --remoteHost=<b>hn1-db-0</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE2</b> 
   </code></pre>

## <a name="create-sap-hana-cluster-resources"></a>Creación de recursos de clúster de SAP HANA

Primero, cree la topología de HANA. Ejecute los comandos siguientes en uno de los nodos del clúster de Pacemaker:

<pre><code>sudo crm configure property maintenance-mode=true

# Replace the bold string with your instance number and HANA system ID

sudo crm configure primitive rsc_SAPHanaTopology_<b>HN1</b>_HDB<b>03</b> ocf:suse:SAPHanaTopology \
  operations \$id="rsc_sap2_<b>HN1</b>_HDB<b>03</b>-operations" \
  op monitor interval="10" timeout="600" \
  op start interval="0" timeout="600" \
  op stop interval="0" timeout="300" \
  params SID="<b>HN1</b>" InstanceNumber="<b>03</b>"

sudo crm configure clone cln_SAPHanaTopology_<b>HN1</b>_HDB<b>03</b> rsc_SAPHanaTopology_<b>HN1</b>_HDB<b>03</b> \
  meta clone-node-max="1" target-role="Started" interleave="true"
</code></pre>

A continuación, cree los recursos de HANA:

> [!IMPORTANT]
> Pruebas recientes han mostrado situaciones en las que netcat deja de responder a las solicitudes debido al trabajo pendiente y a su limitación para controlar solo una conexión. El recurso netcat deja de escuchar las solicitudes del equilibrador de carga de Azure y la dirección IP flotante deja de estar disponible.  
> En el caso de los clústeres de Pacemaker existentes, en el pasado se recomendaba reemplazar netcat por socat. Actualmente se recomienda usar el agente de recursos azure-lb, que forma parte de los agentes de recursos de paquetes, con los siguientes requisitos de versión de paquete:
> - En el caso de SLES 12 SP4/SP5, la versión debe ser, al menos, resource-agents-4.3.018.a7fb5035-3.30.1.  
> - Para SLES 15/15 SP1, la versión debe ser al menos resource-agents-4.3.0184.6ee15eb2-4.13.1.  
>
> Tenga en cuenta que el cambio requerirá un breve tiempo de inactividad.  
> En el caso de los clústeres de Pacemaker existentes, si la configuración ya se ha cambiado para usar socat, como se describe en [Protección de la detección del equilibrador de carga de Azure](https://www.suse.com/support/kb/doc/?id=7024128), no hay ningún requisito para cambiar inmediatamente al agente de recursos azure-lb.

<pre><code># Replace the bold string with your instance number, HANA system ID, and the front-end IP address of the Azure load balancer. 

sudo crm configure primitive rsc_SAPHana_<b>HN1</b>_HDB<b>03</b> ocf:suse:SAPHana \
  operations \$id="rsc_sap_<b>HN1</b>_HDB<b>03</b>-operations" \
  op start interval="0" timeout="3600" \
  op stop interval="0" timeout="3600" \
  op promote interval="0" timeout="3600" \
  op monitor interval="60" role="Master" timeout="700" \
  op monitor interval="61" role="Slave" timeout="700" \
  params SID="<b>HN1</b>" InstanceNumber="<b>03</b>" PREFER_SITE_TAKEOVER="true" \
  DUPLICATE_PRIMARY_TIMEOUT="7200" AUTOMATED_REGISTER="false"

sudo crm configure ms msl_SAPHana_<b>HN1</b>_HDB<b>03</b> rsc_SAPHana_<b>HN1</b>_HDB<b>03</b> \
  meta notify="true" clone-max="2" clone-node-max="1" \
  target-role="Started" interleave="true"

sudo crm configure primitive rsc_ip_<b>HN1</b>_HDB<b>03</b> ocf:heartbeat:IPaddr2 \
  meta target-role="Started" \
  operations \$id="rsc_ip_<b>HN1</b>_HDB<b>03</b>-operations" \
  op monitor interval="10s" timeout="20s" \
  params ip="<b>10.0.0.13</b>"

sudo crm configure primitive rsc_nc_<b>HN1</b>_HDB<b>03</b> azure-lb port=625<b>03</b> \
  meta resource-stickiness=0

sudo crm configure group g_ip_<b>HN1</b>_HDB<b>03</b> rsc_ip_<b>HN1</b>_HDB<b>03</b> rsc_nc_<b>HN1</b>_HDB<b>03</b>

sudo crm configure colocation col_saphana_ip_<b>HN1</b>_HDB<b>03</b> 4000: g_ip_<b>HN1</b>_HDB<b>03</b>:Started \
  msl_SAPHana_<b>HN1</b>_HDB<b>03</b>:Master  

sudo crm configure order ord_SAPHana_<b>HN1</b>_HDB<b>03</b> Optional: cln_SAPHanaTopology_<b>HN1</b>_HDB<b>03</b> \
  msl_SAPHana_<b>HN1</b>_HDB<b>03</b>

# Clean up the HANA resources. The HANA resources might have failed because of a known issue.
sudo crm resource cleanup rsc_SAPHana_<b>HN1</b>_HDB<b>03</b>

sudo crm configure property maintenance-mode=false
sudo crm configure rsc_defaults resource-stickiness=1000
sudo crm configure rsc_defaults migration-threshold=5000
</code></pre>

Asegúrese de que el estado del clúster sea el correcto y de que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.

<pre><code>sudo crm_mon -r

# Online: [ hn1-db-0 hn1-db-1 ]
#
# Full list of resources:
#
# stonith-sbd     (stonith:external/sbd): Started hn1-db-0
# rsc_st_azure    (stonith:fence_azure_arm):      Started hn1-db-1
# Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
#     Started: [ hn1-db-0 hn1-db-1 ]
# Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
#     Masters: [ hn1-db-0 ]
#     Slaves: [ hn1-db-1 ]
# Resource Group: g_ip_HN1_HDB03
#     rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
#     rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
</code></pre>

## <a name="test-the-cluster-setup"></a>Prueba de la configuración del clúster

En esta sección se describe cómo se puede probar la configuración. En todas las pruebas se supone que usted es la raíz y que el sistema maestro de SAP HANA se ejecuta en la máquina virtual **hn1-db-0**.

### <a name="test-the-migration"></a>Prueba de la migración

Antes de comenzar la prueba, asegúrese de que Pacemaker no tenga acciones con error (mediante crm_mon -r), no haya restricciones de ubicación inesperadas (por ejemplo, restos de una prueba de migración) y que HANA se encuentre en estado de sincronización, por ejemplo con SAPHanaSR-showAttr:

<pre><code>hn1-db-0:~ # SAPHanaSR-showAttr

Global cib-time
--------------------------------
global Mon Aug 13 11:26:04 2018

Hosts    clone_state lpa_hn1_lpt node_state op_mode   remoteHost    roles                            score site  srmode sync_state version                vhost
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
hn1-db-0 PROMOTED    1534159564  online     logreplay nws-hana-vm-1 4:P:master1:master:worker:master 150   SITE1 sync   PRIM       2.00.030.00.1522209842 nws-hana-vm-0
hn1-db-1 DEMOTED     30          online     logreplay nws-hana-vm-0 4:S:master1:master:worker:master 100   SITE2 sync   SOK        2.00.030.00.1522209842 nws-hana-vm-1
</code></pre>

Puede migrar el nodo maestro de SAP HANA con el siguiente comando:

<pre><code>crm resource migrate msl_SAPHana_<b>HN1</b>_HDB<b>03</b> <b>hn1-db-1</b>
</code></pre>

Si estableció `AUTOMATED_REGISTER="false"`, esta secuencia de comandos migrará el nodo maestro de SAP HANA y el grupo que contiene la dirección IP virtual a hn1-db-1.

Una vez realizada la migración, la salida de crm_mon -r se parece a esta:

<pre><code>Online: [ hn1-db-0 hn1-db-1 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started hn1-db-1
 Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
     Started: [ hn1-db-0 hn1-db-1 ]
 Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
     Masters: [ hn1-db-1 ]
     Stopped: [ hn1-db-0 ]
 Resource Group: g_ip_HN1_HDB03
     rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
     rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1

Failed Actions:
* rsc_SAPHana_HN1_HDB03_start_0 on hn1-db-0 'not running' (7): call=84, status=complete, exitreason='none',
    last-rc-change='Mon Aug 13 11:31:37 2018', queued=0ms, exec=2095ms
</code></pre>

Se producirá un error al iniciarse el recurso de SAP HANA en hn1-db-0 como secundario. En este caso, configure la instancia de HANA como secundaria mediante la ejecución de este comando:

<pre><code>su - <b>hn1</b>adm

# Stop the HANA instance just in case it is running
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> sapcontrol -nr <b>03</b> -function StopWait 600 10
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=<b>hn1-db-1</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE1</b>
</code></pre>

La migración crea restricciones de ubicación que deben eliminarse de nuevo:

<pre><code># Switch back to root and clean up the failed state
exit
hn1-db-0:~ # crm resource unmigrate msl_SAPHana_<b>HN1</b>_HDB<b>03</b>
</code></pre>

También debe limpiar el estado del recurso de nodo secundario:

<pre><code>hn1-db-0:~ # crm resource cleanup msl_SAPHana_<b>HN1</b>_HDB<b>03</b> <b>hn1-db-0</b>
</code></pre>

Supervise el estado del recurso de HANA mediante crm_mon - r. Una vez iniciado HANA en hn1-db-0, la salida se parecerá a esta:

<pre><code>Online: [ hn1-db-0 hn1-db-1 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started hn1-db-1
 Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
     Started: [ hn1-db-0 hn1-db-1 ]
 Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
     Masters: [ hn1-db-1 ]
     Slaves: [ hn1-db-0 ]
 Resource Group: g_ip_HN1_HDB03
     rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
     rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1
</code></pre>

### <a name="test-the-azure-fencing-agent-not-sbd"></a>Prueba del agente de delimitación de Azure (no SBD)

Puede probar la configuración del agente de delimitación de Azure si deshabilita la interfaz de red en el nodo hn1-db-0:

<pre><code>sudo ifdown eth0
</code></pre>

Ahora, la máquina virtual se reiniciará o se detendrá, según la configuración del clúster.
Si estableció el valor `stonith-action` como desactivado, la máquina virtual se detendrá y los recursos se migrarán a la máquina virtual en ejecución.

Después de volver a iniciar la máquina virtual, el recurso de SAP HANA no se podrá iniciar como secundario si estableció `AUTOMATED_REGISTER="false"`. En este caso, configure la instancia de HANA como secundaria mediante la ejecución de este comando:

<pre><code>su - <b>hn1</b>adm

# Stop the HANA instance just in case it is running
sapcontrol -nr <b>03</b> -function StopWait 600 10
hdbnsutil -sr_register --remoteHost=<b>hn1-db-1</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE1</b>

# Switch back to root and clean up the failed state
exit
crm resource cleanup msl_SAPHana_<b>HN1</b>_HDB<b>03</b> <b>hn1-db-0</b>
</code></pre>

### <a name="test-sbd-fencing"></a>Prueba de delimitación de SBD

Para probar la configuración de SBD, termine el proceso inquisidor.

<pre><code>hn1-db-0:~ # ps aux | grep sbd
root       1912  0.0  0.0  85420 11740 ?        SL   12:25   0:00 sbd: inquisitor
root       1929  0.0  0.0  85456 11776 ?        SL   12:25   0:00 sbd: watcher: /dev/disk/by-id/scsi-360014056f268462316e4681b704a9f73 - slot: 0 - uuid: 7b862dba-e7f7-4800-92ed-f76a4e3978c8
root       1930  0.0  0.0  85456 11776 ?        SL   12:25   0:00 sbd: watcher: /dev/disk/by-id/scsi-360014059bc9ea4e4bac4b18808299aaf - slot: 0 - uuid: 5813ee04-b75c-482e-805e-3b1e22ba16cd
root       1931  0.0  0.0  85456 11776 ?        SL   12:25   0:00 sbd: watcher: /dev/disk/by-id/scsi-36001405b8dddd44eb3647908def6621c - slot: 0 - uuid: 986ed8f8-947d-4396-8aec-b933b75e904c
root       1932  0.0  0.0  90524 16656 ?        SL   12:25   0:00 sbd: watcher: Pacemaker
root       1933  0.0  0.0 102708 28260 ?        SL   12:25   0:00 sbd: watcher: Cluster
root      13877  0.0  0.0   9292  1572 pts/0    S+   12:27   0:00 grep sbd

hn1-db-0:~ # kill -9 1912
</code></pre>

El nodo del clúster hn1-db-0 se reiniciará. Es posible que después el servicio Pacemaker no se inicie. Asegúrese de iniciarlo de nuevo.

### <a name="test-a-manual-failover"></a>Prueba de una conmutación por error manual

Para probar una conmutación por error manual, detenga el servicio `pacemaker` en el nodo hn1-db-0:

<pre><code>service pacemaker stop
</code></pre>

Después de la conmutación por error, puede reiniciar el servicio. Si establece `AUTOMATED_REGISTER="false"`, el recurso de SAP HANA en el nodo hn1-db-0 no se podrá iniciar como secundario. En este caso, configure la instancia de HANA como secundaria mediante la ejecución de este comando:

<pre><code>service pacemaker start
su - <b>hn1</b>adm

# Stop the HANA instance just in case it is running
sapcontrol -nr <b>03</b> -function StopWait 600 10
hdbnsutil -sr_register --remoteHost=<b>hn1-db-1</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE1</b> 

# Switch back to root and clean up the failed state
exit
crm resource cleanup msl_SAPHana_<b>HN1</b>_HDB<b>03</b> <b>hn1-db-0</b>
</code></pre>

### <a name="suse-tests"></a>Pruebas de SUSE

> [!IMPORTANT]
> Asegúrese de que el sistema operativo seleccionado está certificado por SAP para SAP HANA en los tipos específicos de máquina virtual que use. La lista de tipos de máquina virtual certificados para SAP HANA y las versiones de sistema operativo correspondientes se puede consultar en [Plataformas de IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). No olvide consultar la información del tipo de máquina virtual mostrado para obtener una lista completa de las versiones de sistema operativo que SAP HANA admite para ese tipo específico de máquina virtual.

Ejecute todos los casos de prueba que se indican en la guía del escenario optimizado para el rendimiento de SR de SAP HANA o del escenario optimizado para los costos de SR de SAP HANA, en función de su caso de uso. Puede encontrar estas guías en la [página de procedimientos recomendados de SLES para SAP][sles-for-sap-bp].

Las siguientes pruebas son una copia de las descripciones de pruebas de la guía del escenario optimizado para el rendimiento de SR de SAP HANA de SUSE Linux Enterprise Server para SAP Applications 12 SP1. Para obtener una versión actualizada, siempre lea también la propia guía. Antes de inicia la prueba, asegúrese de que HANA esté sincronizado y de que la configuración de Pacemaker sea correcta.

En las siguientes descripciones de pruebas se da por hecho que PREFER_SITE_TAKEOVER = "true" y AUTOMATED_REGISTER = "false".
NOTA:  Las siguientes pruebas están diseñadas para ejecutarse en secuencia y dependen del estado de salida de las pruebas anteriores.

1. PRUEBA 1: DETENCIÓN DE LA BASE DE DATOS PRINCIPAL EN EL NODO 1

   Estado del recurso antes de iniciar la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

   Ejecute los siguientes comandos como <hanasid\>adm en el nodo hn1-db-0:

   <pre><code>hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> HDB stop
   </code></pre>

   Pacemaker detectará la instancia de HANA detenida y conmutará por error al otro nodo detenido. Una vez que se realiza la conmutación por error, la instancia de HANA en el nodo hn1-db-0 se detiene porque Pacemaker no registra automáticamente el nodo como secundario de HANA.

   Ejecute los siguientes comandos para registrar el nodo hn1-db-0 como secundario y limpiar el recurso con error.

   <pre><code>hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-1 --remoteInstance=03 --replicationMode=sync --name=SITE1
   
   # run as root
   hn1-db-0:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-0
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1
   </code></pre>

1. PRUEBA 2: DETENCIÓN DE LA BASE DE DATOS PRINCIPAL EN EL NODO 2

   Estado del recurso antes de iniciar la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1
   </code></pre>

   Ejecute los siguientes comandos como <hanasid\>adm en el nodo hn1-db-1:

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> HDB stop
   </code></pre>

   Pacemaker detectará la instancia de HANA detenida y conmutará por error al otro nodo detenido. Una vez que se realiza la conmutación por error, la instancia de HANA en el nodo hn1-db-1 se detiene porque Pacemaker no registra automáticamente el nodo como secundario de HANA.

   Ejecute los siguientes comandos para registrar el nodo hn1-db-1 como secundario y limpiar el recurso con error.

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-0 --remoteInstance=03 --replicationMode=sync --name=SITE2
   
   # run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

1. PRUEBA 3: BLOQUEO DE LA BASE DE DATOS PRINCIPAL EN EL NODO

   Estado del recurso antes de iniciar la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

   Ejecute los siguientes comandos como <hanasid\>adm en el nodo hn1-db-0:

   <pre><code>hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> HDB kill-9
   </code></pre>
   
   Pacemaker detectará la instancia de HANA terminada y conmutará por error al otro nodo. Una vez que se realiza la conmutación por error, la instancia de HANA en el nodo hn1-db-0 se detiene porque Pacemaker no registra automáticamente el nodo como secundario de HANA.

   Ejecute los siguientes comandos para registrar el nodo hn1-db-0 como secundario y limpiar el recurso con error.

   <pre><code>hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-1 --remoteInstance=03 --replicationMode=sync --name=SITE1
   
   # run as root
   hn1-db-0:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-0
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1
   </code></pre>

1. PRUEBA 4: BLOQUEO DE LA BASE DE DATOS PRINCIPAL EN EL NODO 2

   Estado del recurso antes de iniciar la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1
   </code></pre>

   Ejecute los siguientes comandos como <hanasid\>adm en el nodo hn1-db-1:

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> HDB kill-9
   </code></pre>

   Pacemaker detectará la instancia de HANA terminada y conmutará por error al otro nodo. Una vez que se realiza la conmutación por error, la instancia de HANA en el nodo hn1-db-1 se detiene porque Pacemaker no registra automáticamente el nodo como secundario de HANA.

   Ejecute los siguientes comandos para registrar el nodo hn1-db-1 como secundario y limpiar el recurso con error.

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-0 --remoteInstance=03 --replicationMode=sync --name=SITE2
   
   # run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

1. PRUEBA 5: BLOQUEO DEL NODO DE SITIO PRINCIPAL (NODO 1)

   Estado del recurso antes de iniciar la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

   Ejecute los siguientes comandos como raíz en el nodo hn1-db-0:

   <pre><code>hn1-db-0:~ #  echo 'b' > /proc/sysrq-trigger
   </code></pre>

   Pacemaker detectará el nodo del clúster terminado y lo delimitará. Una vez que el nodo está delimitado, Pacemaker desencadenará una adquisición de la instancia de HANA. Cuando se reinicia el nodo delimitado, Pacemaker no se inicia automáticamente.

   Ejecute los siguientes comandos para iniciar Pacemaker, limpiar los mensajes de SBD del nodo hn1-db-0, registrar el nodo hn1-db-0 como secundario y limpiar el recurso con error.

   <pre><code># run as root
   # list the SBD device(s)
   hn1-db-0:~ # cat /etc/sysconfig/sbd | grep SBD_DEVICE=
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116;/dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1;/dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3"
   
   hn1-db-0:~ # sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116 -d /dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1 -d /dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3 message hn1-db-0 clear
   
   hn1-db-0:~ # systemctl start pacemaker
   
   # run as &lt;hanasid&gt;adm
   hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-1 --remoteInstance=03 --replicationMode=sync --name=SITE1
   
   # run as root
   hn1-db-0:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-0
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1
   </code></pre>

1. PRUEBA 6: BLOQUEO DEL NODO DE SITIO SECUNDARIO (NODO 2)

   Estado del recurso antes de iniciar la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-1
   </code></pre>

   Ejecute los siguientes comandos como raíz en el nodo hn1-db-1:

   <pre><code>hn1-db-1:~ #  echo 'b' > /proc/sysrq-trigger
   </code></pre>

   Pacemaker detectará el nodo del clúster terminado y lo delimitará. Una vez que el nodo está delimitado, Pacemaker desencadenará una adquisición de la instancia de HANA. Cuando se reinicia el nodo delimitado, Pacemaker no se inicia automáticamente.

   Ejecute los siguientes comandos para iniciar Pacemaker, limpiar los mensajes de SBD del nodo hn1-db-1, registrar el nodo hn1-db-1 como secundario y limpiar el recurso con error.

   <pre><code># run as root
   # list the SBD device(s)
   hn1-db-1:~ # cat /etc/sysconfig/sbd | grep SBD_DEVICE=
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116;/dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1;/dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3"
   
   hn1-db-1:~ # sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116 -d /dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1 -d /dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3 message hn1-db-1 clear
   
   hn1-db-1:~ # systemctl start pacemaker
   
   # run as &lt;hanasid&gt;adm
   hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-0 --remoteInstance=03 --replicationMode=sync --name=SITE2
   
   # run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

1. PRUEBA 7: DETENCIÓN DE LA BASE DE DATOS SECUNDARIA EN EL NODO 2

   Estado del recurso antes de iniciar la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

   Ejecute los siguientes comandos como <hanasid\>adm en el nodo hn1-db-1:

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> HDB stop
   </code></pre>

   Pacemaker detectará la instancia de HANA detenida y marcará el recurso como erróneo en el nodo hn1-db-1. Pacemaker reiniciará automáticamente la instancia de HANA. Ejecute el siguiente comando para limpiar el estado de error.

   <pre><code># run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

1. PRUEBA 8: BLOQUEO DE LA BASE DE DATOS SECUNDARIA EN EL NODO 2

   Estado del recurso antes de iniciar la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

   Ejecute los siguientes comandos como <hanasid\>adm en el nodo hn1-db-1:

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> HDB kill-9
   </code></pre>

   Pacemaker detectará la instancia de HANA terminada y marcará el recurso como erróneo en el nodo hn1-db-1. Ejecute el siguiente comando para limpiar el estado de error. Pacemaker reiniciará entonces automáticamente la instancia de HANA.

   <pre><code># run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

1. PRUEBA 9: BLOQUEO DEL NODO DE SITIO SECUNDARIO (NODO 2) QUE EJECUTA LA BASE DE DATOS DE HANA SECUNDARIA

   Estado del recurso antes de iniciar la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

   Ejecute los siguientes comandos como raíz en el nodo hn1-db-1:

   <pre><code>hn1-db-1:~ # echo b > /proc/sysrq-trigger
   </code></pre>

   Pacemaker detectará el nodo del clúster terminado y lo delimitará. Cuando se reinicia el nodo delimitado, Pacemaker no se inicia automáticamente.

   Ejecute los siguientes comandos para iniciar Pacemaker, limpiar los mensajes de SBD del nodo hn1-db-1 y limpiar el recurso con error.

   <pre><code># run as root
   # list the SBD device(s)
   hn1-db-1:~ # cat /etc/sysconfig/sbd | grep SBD_DEVICE=
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116;/dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1;/dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3"
   
   hn1-db-1:~ # sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116 -d /dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1 -d /dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3 message hn1-db-1 clear
   
   hn1-db-1:~ # systemctl start pacemaker  
   
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:azure-lb):      Started hn1-db-0
   </code></pre>

## <a name="next-steps"></a>Pasos siguientes

* [Planeamiento e implementación de Azure Virtual Machines para SAP][planning-guide]
* [Implementación de Azure Virtual Machines para SAP][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP][dbms-guide]
