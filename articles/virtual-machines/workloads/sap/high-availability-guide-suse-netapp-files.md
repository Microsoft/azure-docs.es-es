---
title: Alta disponibilidad de máquinas virtuales de Azure para SAP NW en SLES con Azure NetApp Files | Microsoft Docs
description: Guía de alta disponibilidad de SAP NetWeaver en SUSE Linux Enterprise Server con Azure NetApp Files para las aplicaciones de SAP
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
ms.openlocfilehash: 75573cb49bcc0aca206ad8f13f060121e70a0621
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87760624"
---
# <a name="high-availability-for-sap-netweaver-on-azure-vms-on-suse-linux-enterprise-server-with-azure-netapp-files-for-sap-applications"></a>Alta disponibilidad de SAP NetWeaver en VM de Azure en SUSE Linux Enterprise Server con Azure NetApp Files para las aplicaciones de SAP

[dbms-guide]:dbms_guide_general.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-azure-doc]:https://docs.microsoft.com/azure/azure-netapp-files/
[anf-avail-matrix]:https://azure.microsoft.com/global-infrastructure/services/?products=storage&regions=all
[anf-register]:https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register
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

En este artículo se describe cómo implementar y configurar las máquinas virtuales, y cómo instalar el marco del clúster y un sistema SAP NetWeaver 7.50 de alta disponibilidad, mediante [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md).
En las configuraciones de ejemplo, los comandos de instalación. etc., la instancia de ASCS es 00, la instancia de ERS es 01, la instancia de aplicación principal (PAS) es 02 y la instancia de aplicación (AAS) es 03. Se usan los controles de calidad de los identificadores del sistema SAP. 

En este artículo se explica cómo lograr una alta disponibilidad para la aplicación de SAP NetWeaver con Azure NetApp Files. El nivel de la base de datos no se trata en detalle en este artículo.

Lea primero las notas y los documentos de SAP siguientes:

* [Documentación sobre Azure NetApp Files][anf-azure-doc] 
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
* La WIKI de la comunidad SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contiene todas las notas de SAP que se necesitan para Linux.
* [Planeación e implementación de Azure Virtual Machines para SAP en Linux][planning-guide]
* [Implementación de Azure Virtual Machines para SAP en Linux][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP en Linux][dbms-guide]
* [Guías de procedimientos recomendados de SUSE SAP HA][suse-ha-guide] Las guías contienen toda la información necesaria para la configuración de Netweaver HA y la replicación del sistema de SAP HANA en el entorno local. Use estas guías como base de referencia general. Proporcionan información mucho más detallada.
* [Notas de la versión de la Extensión 12 SP3 de alta disponibilidad para SUSE][suse-ha-12sp3-relnotes]
* [Aplicaciones de NetApp SAP en Microsoft Azure que usan Azure NetApp Files][anf-sap-applications-azure]

## <a name="overview"></a>Información general

La alta disponibilidad de los servicios centrales de SAP Netweaver requiere almacenamiento compartido.
Para lograrlo en SUSE Linux hasta ahora era necesario crear el clúster independiente NFS de alta disponibilidad. 

Ahora ya es posible lograr alta disponibilidad en SAP NetWeaver mediante el uso de almacenamiento compartido implementado en Azure NetApp Files. Mediante el uso de Azure NetApp Files para el almacenamiento compartido se elimina la necesidad del [clúster NFS](./high-availability-guide-suse-nfs.md) adicional. Pacemaker sigue siendo necesario para lograr una alta disponibilidad de los servicios centrales de SAP NetWeaver (ASCS/SCS).


![Información general sobre la alta disponibilidad de SAP NetWeaver](./media/high-availability-guide-suse-anf/high-availability-guide-suse-anf.png)

SAP NetWeaver ASCS, SAP NetWeaver SCS, SAP NetWeaver ERS y la base de datos SAP HANA usan direcciones IP virtuales y el nombre de host virtual. En Azure, se requiere un [equilibrador de carga](../../../load-balancer/load-balancer-overview.md) para usar una dirección IP virtual. Se recomienda usar [Standard Load Balancer](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md). En la lista siguiente se muestra la configuración del equilibrador de carga (A)SCS y ERS.

### <a name="ascs"></a>(A)SCS

* Configuración de front-end
  * Dirección IP 10.1.1.20.
* Puerto de sondeo
  * Puerto 620<strong>&lt;nr&gt;</strong>
* Reglas de equilibrio de carga.
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
  * Dirección IP 10.1.1.21.
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


## <a name="setting-up-the-azure-netapp-files-infrastructure"></a>Configuración de la infraestructura de Azure NetApp Files 

SAP NetWeaver requiere un almacenamiento compartido para el directorio de transporte y perfil.  Antes de continuar con la configuración de la infraestructura de Azure NetApp Files, familiarícese con la [documentación correspondiente][anf-azure-doc]. Compruebe si la región de Azure seleccionada ofrece Azure NetApp Files. El siguiente vínculo muestra la disponibilidad de Azure NetApp Files por región de Azure: [Disponibilidad de Azure NetApp Files por región de Azure][anf-avail-matrix].

Azure NetApp Files está disponible en varias [regiones de Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp). Antes de implementar Azure NetApp Files, solicite la incorporación a este siguiendo las [instrucciones para registrarse en Azure NetApp Files][anf-register]. 

### <a name="deploy-azure-netapp-files-resources"></a>Implementación de recursos de Azure NetApp Files  

En estos pasos se supone que ya ha implementado [Azure Virtual Network](../../../virtual-network/virtual-networks-overview.md). Los recursos de Azure NetApp Files y las máquinas virtuales en las que esos recursos se montarán se deben implementar en la misma red virtual de Azure o en redes virtuales emparejadas.  

1. Si aún no lo ha hecho, solicite la [incorporación a Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).  

2. Cree la cuenta de NetApp en la región de Azure seleccionada mediante las [instrucciones para crear la cuenta de NetApp](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md).  
3. Configure el grupo de capacidad de Azure NetApp Files con las [instrucciones correspondientes](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md).  
La arquitectura de SAP Netweaver que se presenta en este artículo usa un único grupo de capacidad de Azure NetApp Files, SKU Premium. Se recomienda usar SKU Premium de Azure NetApp Files para cargas de trabajo de aplicaciones de SAP NetWeaver en Azure.  

4. Delegue una subred en Azure NetApp Files como se describe en las [instrucciones correspondientes](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).  

5. Implemente los volúmenes de Azure NetApp Files mediante las [instrucciones correspondientes](../../../azure-netapp-files/azure-netapp-files-create-volumes.md). Implemente los volúmenes en la [subred](/rest/api/virtualnetwork/subnets) de Azure NetApp Files designada. Las direcciones IP de los volúmenes de Azure NetApp se asignan automáticamente. Tenga en cuenta que los recursos de Azure NetApp Files y las máquinas virtuales de Azure deben estar en la misma red virtual de Azure o en redes virtuales emparejadas. En este ejemplo se usan dos volúmenes de Azure NetApp Files: sap<b>QAS</b> y trans. Las rutas de acceso de los archivos que se montan en los correspondientes puntos de montaje son /usrsap<b>qas</b>/sapmnt<b>QAS</b>, /usrsap<b>qas</b>/usrsap<b>QAS</b>sys, etc.  

   1. volumen sap<b>QAS</b> (nfs://10.1.0.4/usrsap<b>qas</b>/sapmnt<b>QAS</b>)
   2. volumen sap<b>QAS</b> (nfs://10.1.0.4/usrsap<b>qas</b>/usrsap<b>QAS</b>ascs)
   3. volumen sap<b>QAS</b> (nfs://10.1.0.4/usrsap<b>qas</b>/usrsap<b>QAS</b>sys)
   4. volumen sap<b>QAS</b> (nfs://10.1.0.4/usrsap<b>qas</b>/usrsap<b>QAS</b>ers)
   5. volumen trans (nfs://10.1.0.4/trans)
   6. volumen sap<b>QAS</b> (nfs://10.1.0.4/usrsap<b>qas</b>/usrsap<b>QAS</b>pas)
   7. volumen sap<b>QAS</b> (nfs://10.1.0.4/usrsap<b>qas</b>/usrsap<b>QAS</b>aas)

   
En este ejemplo, hemos usado Azure NetApp Files para todos los sistemas de archivos de SAP Netweaver para demostrar cómo se puede usar Azure NetApp Files. Los sistemas de archivos SAP que no es necesario montar mediante NFS se pueden también implementar en [Azure Disk Storage](../../windows/disks-types.md#premium-ssd). En este ejemplo, <b>a-e</b> deben estar en Azure NetApp Files y <b>f-g</b> (es decir, /usr/sap/<b>QAS</b>/D<b>02</b>, /usr/sap/<b>QAS</b>/D<b>03</b>) se podrían implementar en Azure Disk Storage. 

### <a name="important-considerations"></a>Consideraciones importantes

A la hora de considerar Azure NetApp Files para SAP Netweaver en la arquitectura de alta disponibilidad de SUSE, tenga en cuenta los siguientes aspectos importantes:

- El grupo de capacidad mínimo es de 4 TiB. El tamaño del grupo de capacidad puede aumentar en incrementos de 1 TiB.
- El volumen mínimo es de 100 GiB.
- Azure NetApp Files y todas las máquinas virtuales en las que los volúmenes de Azure NetApp Files se montarán, se deben implementar en la misma red virtual de Azure o en [redes virtuales emparejadas](../../../virtual-network/virtual-network-peering-overview.md) de la misma región. Ahora también se admite el acceso de Azure NetApp Files mediante emparejamiento de red virtual en la misma región. Aún no se admite el acceso de Azure NetApp a través del emparejamiento global.
- La red virtual seleccionada debe tener una subred delegada en Azure NetApp Files.
- Azure NetApp Files ofrece la [directiva de exportación](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md): puede controlar los clientes permitidos, el tipo de acceso (lectura y escritura, solo lectura, etc.). 
- La característica Azure NetApp Files no depende aún de la zona. En la actualidad, la característica Azure NetApp Files no se implementa en todas las zonas de disponibilidad de una región de Azure. Tenga en cuenta las posibles implicaciones de latencia en algunas regiones de Azure. 
- Los volúmenes de Azure NetApp Files se pueden implementar como volúmenes NFSv3 o NFSv4.1. Los dos protocolos son compatibles con el nivel de aplicación de SAP (ASCS/ERS, servidores de aplicaciones de SAP). 

## <a name="deploy-linux-vms-manually-via-azure-portal"></a>Implementación manual de VM de Linux mediante Azure Portal

Primero deberá crear los volúmenes de Azure NetApp Files. Implemente las VM. Después, creará un equilibrador de carga y usará las máquinas virtuales de los grupos de servidores back-end.

1. Creación de un grupo de recursos
1. Creación de una red virtual
1. Cree un conjunto de disponibilidad para ACS.  
   Establecimiento del dominio máximo de actualización
1. Creación de la máquina virtual 1  
   Use al menos SLES4SAP 12 SP3, en este ejemplo se usa la imagen SLES4SAP 12 SP3.  
   Seleccione el conjunto de disponibilidad creado anteriormente para ASCS.  
1. Creación de la máquina virtual 2  
   Use al menos SLES4SAP 12 SP3, en este ejemplo se usa la imagen SLES4SAP 12 SP3.  
   Seleccione el conjunto de disponibilidad creado anteriormente para ASCS.  
1. Cree un conjunto de disponibilidad para las instancias de aplicaciones de SAP (PAS, AAS).    
   Establecimiento del dominio máximo de actualización
1. Cree la máquina virtual 3.  
   Use al menos SLES4SAP 12 SP3, en este ejemplo se usa la imagen SLES4SAP 12 SP3.  
   Seleccione el conjunto de disponibilidad creado anteriormente para PAS/AAS.   
1. Cree la máquina virtual 4.  
   Use al menos SLES4SAP 12 SP3, en este ejemplo se usa la imagen SLES4SAP 12 SP3.  
   Seleccione el conjunto de disponibilidad creado anteriormente para PAS/AAS.  

## <a name="disable-id-mapping-if-using-nfsv41"></a>Deshabilite la asignación de identificadores (si se usa NFSv4.1)

Las instrucciones de esta sección solo se aplican si se usan volúmenes de Azure NetApp Files con el protocolo NFSv4.1. Realice la configuración en todas las máquinas virtuales donde se montarán los volúmenes NFSv4.1 de Azure NetApp Files.  

1. Compruebe la configuración del dominio NFS. Asegúrese de que el dominio esté configurado como dominio predeterminado de Azure NetApp Files, es decir, **`defaultv4iddomain.com`** y que la asignación se haya establecido en **nobody**.  

    > [!IMPORTANT]
    > Asegúrese de establecer el dominio NFS de `/etc/idmapd.conf` en la máquina virtual para que coincida con la configuración de dominio predeterminada en Azure NetApp Files: **`defaultv4iddomain.com`** . Si hay alguna discrepancia entre la configuración de dominio del cliente NFS (es decir, la máquina virtual) y el servidor NFS (es decir la configuración de Azure NetApp), los permisos de archivos en volúmenes de Azure NetApp que estén montados en las máquinas virtuales se mostrarán como `nobody`.  

    <pre><code>
    sudo cat /etc/idmapd.conf
    # Example
    [General]
    Verbosity = 0
    Pipefs-Directory = /var/lib/nfs/rpc_pipefs
    Domain = <b>defaultv4iddomain.com</b>
    [Mapping]
    Nobody-User = <b>nobody</b>
    Nobody-Group = <b>nobody</b>
    </code></pre>

4. **[A]** Compruebe `nfs4_disable_idmapping`. Debe establecerse en **S**. Para crear la estructura de directorio en la que se encuentra `nfs4_disable_idmapping`, ejecute el comando mount. No podrá crear manualmente el directorio en /sys/modules, ya que el acceso está reservado para el kernel o los controladores.  

    <pre><code>
    # Check nfs4_disable_idmapping 
    cat /sys/module/nfs/parameters/nfs4_disable_idmapping
    # If you need to set nfs4_disable_idmapping to Y
    mkdir /mnt/tmp
    mount 10.1.0.4:/sapmnt/<b>qas</b> /mnt/tmp
    umount  /mnt/tmp
    echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping
    # Make the configuration permanent
    echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
    </code></pre>


## <a name="setting-up-ascs"></a>Configuración de (A)SCS

En este ejemplo, los recursos se implementaron manualmente mediante [Azure Portal](https://portal.azure.com/#home).

### <a name="deploy-azure-load-balancer-manually-via-azure-portal"></a>Implemente Azure Load Balancer manualmente a través de Azure Portal.

Primero deberá crear los volúmenes de Azure NetApp Files. Implemente las VM. Después, creará un equilibrador de carga y usará las máquinas virtuales del grupo de servidores back-end.

1. Cree un equilibrador de carga (interno, estándar):  
   1. Creación de las direcciones IP de front-end
      1. Dirección IP 10.1.1.20 de ASCS.
         1. Abra el equilibrador de carga, seleccione el grupo de direcciones IP de front-end y haga clic en Agregar
         1. Escriba el nombre del nuevo grupo de direcciones IP de front-end (por ejemplo, **frontend.QAS.ASCS**).
         1. 2\.Establezca Asignación en Estática y escriba la dirección IP (por ejemplo, **10.1.1.20**).
         1. Haga clic en Aceptar
      1. Dirección IP 10.1.1.21 para ASCS ERS.
         * Repita los pasos anteriores a partir de "a" para crear una dirección IP para el ERS (por ejemplo, **10.1.1.21** y **frontend.QAS.ERS**).
   1. Creación del grupo de servidores back-end
      1. Abra el equilibrador de carga, seleccione los grupos de back-end y haga clic en Agregar
      1. Escriba el nombre del nuevo grupo de servidores back-end (por ejemplo, **backend.QAS**).
      1. Haga clic en Agregar una máquina virtual.
      1. Seleccione Máquina virtual.
      1. Seleccione las máquinas virtuales del clúster de (A)SCS y sus direcciones IP.
      1. Haga clic en Agregar
   1. Creación de los sondeos de estado
      1. Puerto 620**00** para ASCS
         1. Abra el equilibrador de carga, seleccione los sondeos de estado y haga clic en Agregar
         1. Escriba el nombre del sondeo de estado nuevo (por ejemplo **health.QAS.ASCS**).
         1. Seleccione TCP como protocolo, puerto 620**00**, y mantenga el intervalo de 5 y el umbral incorrecto 2.
         1. Haga clic en Aceptar
      1. Puerto 621**01** para ASCS ERS.
            * Repita los pasos anteriores a partir de "c" para crear un sondeo de estado para ERS (por ejemplo 621**01** y **health.QAS.ERS**).
   1. Reglas de equilibrio de carga.
      1. Creación de un grupo de servidores back-end para ASCS
         1. Abra el equilibrador de carga, seleccione las reglas de equilibrio de carga y haga clic en Agregar.
         1. Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, **lb.QAS.ASCS**)
         1. Seleccione la dirección IP de front-end para ASCS, el grupo de back-end y el sondeo de estado creados anteriormente (por ejemplo, **frontend.QAS.ASCS**, **backend.QAS** y **health.QAS.ASCS**)
         1. Seleccione **Puertos HA**
         1. Aumente el tiempo de espera de inactividad a 30 minutos
         1. **Asegúrese de habilitar la dirección IP flotante**
         1. Haga clic en Aceptar
         * Repita los pasos anteriores para crear reglas de equilibrio de carga para ERS (por ejemplo **lb.QAS.ERS**)
1. Como alternativa, si el escenario requiere un equilibrador de carga básico (interno), siga estos pasos:  
   1. Creación de las direcciones IP de front-end
      1. Dirección IP 10.1.1.20 de ASCS.
         1. Abra el equilibrador de carga, seleccione el grupo de direcciones IP de front-end y haga clic en Agregar
         1. Escriba el nombre del nuevo grupo de direcciones IP de front-end (por ejemplo, **frontend.QAS.ASCS**).
         1. 2\.Establezca Asignación en Estática y escriba la dirección IP (por ejemplo, **10.1.1.20**).
         1. Haga clic en Aceptar
      1. Dirección IP 10.1.1.21 para ASCS ERS.
         * Repita los pasos anteriores a partir de "a" para crear una dirección IP para el ERS (por ejemplo, **10.1.1.21** y **frontend.QAS.ERS**).
   1. Creación del grupo de servidores back-end
      1. Abra el equilibrador de carga, seleccione los grupos de back-end y haga clic en Agregar
      1. Escriba el nombre del nuevo grupo de servidores back-end (por ejemplo, **backend.QAS**).
      1. Haga clic en Agregar una máquina virtual.
      1. Seleccione el conjunto de disponibilidad que creó anteriormente para ASCS. 
      1. Seleccione las máquinas virtuales del clúster (A)SCS.
      1. Haga clic en Aceptar
   1. Creación de los sondeos de estado
      1. Puerto 620**00** para ASCS
         1. Abra el equilibrador de carga, seleccione los sondeos de estado y haga clic en Agregar
         1. Escriba el nombre del sondeo de estado nuevo (por ejemplo **health.QAS.ASCS**).
         1. Seleccione TCP como protocolo, puerto 620**00**, y mantenga el intervalo de 5 y el umbral incorrecto 2.
         1. Haga clic en Aceptar
      1. Puerto 621**01** para ASCS ERS.
            * Repita los pasos anteriores a partir de "c" para crear un sondeo de estado para ERS (por ejemplo 621**01** y **health.QAS.ERS**).
   1. Reglas de equilibrio de carga.
      1. TCP 32**00** para ASCS
         1. Abra el equilibrador de carga, seleccione las reglas de equilibrio de carga y haga clic en Agregar.
         1. Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, **lb.QAS.ASCS.3200**).
         1. Seleccione la dirección IP de front-end para ASCS, el grupo de servidores back-end y el sondeo de estado que creó anteriormente (por ejemplo, **frontend.QAS.ASCS**).
         1. Conserve el protocolo **TCP** y escriba el puerto **3200**.
         1. Aumente el tiempo de espera de inactividad a 30 minutos
         1. **Asegúrese de habilitar la dirección IP flotante**
         1. Haga clic en Aceptar
      1. Puertos adicionales para ASCS
         * Repita los pasos anteriores para los puertos 36**00**, 39**00**, 81**00**, 5**00**13, 5**00**14, 5**00**16 y TCP para ASCS.
      1. Puertos adicionales para ASCS ERS
         * Repita los pasos anteriores a partir de "d" para los puertos 32**01**, 33**01**, 5**01**13, 5**01**14, 5**01**16 y TCP para ASCS ERS

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

   <pre><code>sudo zypper info sap-suse-cluster-connector
   
    # Information for package sap-suse-cluster-connector:
    # ---------------------------------------------------
    # Repository     : SLE-12-SP3-SAP-Updates
    # Name           : sap-suse-cluster-connector
    # Version        : 3.1.0-8.1
    # Arch           : noarch
    # Vendor         : SUSE LLC &lt;https://www.suse.com/&gt;
    # Support Level  : Level 3
    # Installed Size : 45.6 KiB
    # Installed      : Yes
    # Status         : up-to-date
    # Source package : sap-suse-cluster-connector-3.1.0-8.1.src
    # Summary        : SUSE High Availability Setup for SAP Products
   </code></pre>

2. **[A]** Actualice los agentes de recursos de SAP  
   
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

3. **[A]** Configure la resolución nombres de host

   Puede usar un servidor DNS o modificar /etc/hosts en todos los nodos. En este ejemplo se muestra cómo utilizar el archivo /etc/hosts.
   Reemplace la dirección IP y el nombre de host en los siguientes comandos

   <pre><code>sudo vi /etc/hosts
   </code></pre>

   Inserte las siguientes líneas en /etc/hosts. Cambie la dirección IP y el nombre de host para que coincida con su entorno   

   <pre><code>
   # IP address of cluster node 1
   <b>10.1.1.18    anftstsapcl1</b>
   # IP address of cluster node 2
   <b>10.1.1.6     anftstsapcl2</b>
   # IP address of the load balancer frontend configuration for SAP Netweaver ASCS
   <b>10.1.1.20    anftstsapvh</b>
   # IP address of the load balancer frontend configuration for SAP Netweaver ERS
   <b>10.1.1.21    anftstsapers</b>
   </code></pre>

4. **[1]**  Cree directorios de SAP en el volumen de Azure NetApp Files.  
   Monte el volumen de Azure NetApp Files temporalmente en una de las máquinas virtuales y cree los directorios de SAP (rutas de acceso de archivos).  

   ```
    # mount temporarily the volume
    sudo mkdir -p /saptmp
    # If using NFSv3
    sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=3,tcp 10.1.0.4:/sapQAS /saptmp
    # If using NFSv4.1
    sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys,tcp 10.1.0.4:/sapQAS /saptmp
    # create the SAP directories
    sudo cd /saptmp
    sudo mkdir -p sapmntQAS
    sudo mkdir -p usrsapQASascs
    sudo mkdir -p usrsapQASers
    sudo mkdir -p usrsapQASsys
    sudo mkdir -p usrsapQASpas
    sudo mkdir -p usrsapQASaas
    # unmount the volume and delete the temporary directory
    sudo cd ..
    sudo umount /saptmp
    sudo rmdir /saptmp
    ``` 

## <a name="prepare-for-sap-netweaver-installation"></a>Preparación de la instalación de SAP NetWeaver

1. **[A]** Cree los directorios compartidos

   <pre><code>sudo mkdir -p /sapmnt/<b>QAS</b>
   sudo mkdir -p /usr/sap/trans
   sudo mkdir -p /usr/sap/<b>QAS</b>/SYS
   sudo mkdir -p /usr/sap/<b>QAS</b>/ASCS<b>00</b>
   sudo mkdir -p /usr/sap/<b>QAS</b>/ERS<b>01</b>
   
   sudo chattr +i /sapmnt/<b>QAS</b>
   sudo chattr +i /usr/sap/trans
   sudo chattr +i /usr/sap/<b>QAS</b>/SYS
   sudo chattr +i /usr/sap/<b>QAS</b>/ASCS<b>00</b>
   sudo chattr +i /usr/sap/<b>QAS</b>/ERS<b>01</b>
   </code></pre>

2. **[A]** Configure `autofs`

   <pre><code>
   sudo vi /etc/auto.master
   # Add the following line to the file, save and exit
   /- /etc/auto.direct
   </code></pre>

   Si usa NFSv3, cree un archivo con:

   <pre><code>
   sudo vi /etc/auto.direct
   # Add the following lines to the file, save and exit
   /sapmnt/<b>QAS</b> -nfsvers=3,nobind 10.1.0.4:/usrsap<b>qas</b>/sapmnt<b>QAS</b>
   /usr/sap/trans -nfsvers=3,nobind 10.1.0.4:/trans
   /usr/sap/<b>QAS</b>/SYS -nfsvers=3,nobind 10.1.0.4:/usrsap<b>qas</b>/usrsap<b>QAS</b>sys
   </code></pre>
   
   Si usa NFSv4.1, cree un archivo con:

   <pre><code>
   sudo vi /etc/auto.direct
   # Add the following lines to the file, save and exit
   /sapmnt/<b>QAS</b> -nfsvers=4.1,nobind,sec=sys 10.1.0.4:/usrsap<b>qas</b>/sapmnt<b>QAS</b>
   /usr/sap/trans -nfsvers=4.1,nobind,sec=sys 10.1.0.4:/trans
   /usr/sap/<b>QAS</b>/SYS -nfsvers=4.1,nobind,sec=sys 10.1.0.4:/usrsap<b>qas</b>/usrsap<b>QAS</b>sys
   </code></pre>
   
   > [!NOTE]
   > Asegúrese de hacer coincidir la versión del protocolo NFS de los volúmenes Azure NetApp Files al montar los volúmenes. Si los volúmenes de Azure NetApp Files se crean como volúmenes NFSv3, use la configuración de NFSv3 correspondiente. Si los volúmenes de Azure NetApp Files se crean como volúmenes NFSv4.1, siga las instrucciones para deshabilitar la asignación de Id. y asegúrese de usar la configuración de NFSv4.1 correspondiente. En este ejemplo, los volúmenes de Azure NetApp Files se crearon como volúmenes NFSv3.  
   
   Reinicie `autofs` para montar los recursos compartidos nuevos.
    <pre><code>
      sudo systemctl enable autofs
      sudo service autofs restart
     </code></pre>

3. **[A]** Configure el archivo de intercambio

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

   <pre><code>sudo crm node standby <b>anftstsapcl2</b>
   # If using NFSv3
   sudo crm configure primitive fs_<b>QAS</b>_ASCS Filesystem device='<b>10.1.0.4</b>/usrsap<b>qas</b>/usrsap<b>QAS</b>ascs' directory='/usr/sap/<b>QAS</b>/ASCS<b>00</b>' fstype='nfs' \
     op start timeout=60s interval=0 \
     op stop timeout=60s interval=0 \
     op monitor interval=20s timeout=40s
   
   # If using NFSv4.1
   sudo crm configure primitive fs_<b>QAS</b>_ASCS Filesystem device='<b>10.1.0.4</b>:/usrsap<b>qas</b>/usrsap<b>QAS</b>ascs' directory='/usr/sap/<b>QAS</b>/ASCS<b>00</b>' fstype='nfs' options='sec=sys,vers=4.1' \
     op start timeout=60s interval=0 \
     op stop timeout=60s interval=0 \
     op monitor interval=20s timeout=40s
   
   sudo crm configure primitive vip_<b>QAS</b>_ASCS IPaddr2 \
     params ip=<b>10.1.1.20</b> cidr_netmask=<b>24</b> \
     op monitor interval=10 timeout=20
   
   sudo crm configure primitive nc_<b>QAS</b>_ASCS azure-lb port=620<b>00</b>
   
   sudo crm configure group g-<b>QAS</b>_ASCS fs_<b>QAS</b>_ASCS nc_<b>QAS</b>_ASCS vip_<b>QAS</b>_ASCS \
      meta resource-stickiness=3000
   </code></pre>

   Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.

   <pre><code>sudo crm_mon -r
   
   # Node anftstsapcl2: standby
   # <b>Online: [ anftstsapcl1 ]</b>
   # 
   # Full list of resources:
   #
   # Resource Group: g-QAS_ASCS
   #     fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    <b>Started anftstsapcl1</b>
   #     nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      <b>Started anftstsapcl1</b>
   #     vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       <b>Started anftstsapcl1</b>
   # stonith-sbd     (stonith:external/sbd): <b>Started anftstsapcl2</b>
   </code></pre>
  
2. **[1]** Instale SAP NetWeaver ASCS  

   Instale SAP NetWeaver ASCS como raíz en el primer nodo mediante un nombre de host virtual que se asigna a la dirección IP de la configuración de front-end del equilibrador de carga para ASCS; por ejemplo, <b>anftstsapvh</b>, <b>10.1.1.20</b>. Especifique también el número de instancia que usó para el sondeo del equilibrador de carga; por ejemplo, <b>00</b>.

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst. Puede usar el parámetro SAPINST_USE_HOSTNAME para instalar SAP con el nombre de host virtual.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b> SAPINST_USE_HOSTNAME=<b>virtual_hostname</b>
   </code></pre>

   Si se produce un error en la instalación para crear una subcarpeta en /usr/sap/**QAS**/ASCS**00**, pruebe a establecer el propietario y el grupo de la carpeta ASCS**00** e inténtelo de nuevo. 

   <pre><code>
   chown <b>qas</b>adm /usr/sap/<b>QAS</b>/ASCS<b>00</b>
   chgrp sapsys /usr/sap/<b>QAS</b>/ASCS<b>00</b>
   </code></pre>

3. **[1]** Cree un recurso IP virtual y un sondeo de estado para la instancia de ERS

   <pre><code>
   sudo crm node online <b>anftstsapcl2</b>
   sudo crm node standby <b>anftstsapcl1</b>
   # If using NFSv3
   sudo crm configure primitive fs_<b>QAS</b>_ERS Filesystem device='<b>10.1.0.4</b>:/usrsap<b>qas</b>/usrsap<b>QAS</b>ers' directory='/usr/sap/<b>QAS</b>/ERS<b>01</b>' fstype='nfs' \
     op start timeout=60s interval=0 \
     op stop timeout=60s interval=0 \
     op monitor interval=20s timeout=40s
   
   # If using NFSv4.1
   sudo crm configure primitive fs_<b>QAS</b>_ERS Filesystem device='<b>10.1.0.4</b>:/usrsap<b>qas</b>/usrsap<b>QAS</b>ers' directory='/usr/sap/<b>QAS</b>/ERS<b>01</b>' fstype='nfs' options='sec=sys,vers=4.1'\
     op start timeout=60s interval=0 \
     op stop timeout=60s interval=0 \
     op monitor interval=20s timeout=40s
   
   sudo crm configure primitive vip_<b>QAS</b>_ERS IPaddr2 \
     params ip=<b>10.1.1.21</b> cidr_netmask=<b>24</b> \
     op monitor interval=10 timeout=20
   
   sudo crm configure primitive nc_<b>QAS</b>_ERS azure-lb port=621<b>01</b>
   
   sudo crm configure group g-<b>QAS</b>_ERS fs_<b>QAS</b>_ERS nc_<b>QAS</b>_ERS vip_<b>QAS</b>_ERS
   </code></pre>

   Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.

   <pre><code>sudo crm_mon -r
   
   # Node <b>anftstsapcl1: standby</b>
   # <b>Online: [ anftstsapcl2 ]</b>
   # 
   # Full list of resources:
   #
   # stonith-sbd     (stonith:external/sbd): <b>Started anftstsapcl2</b>
   #  Resource Group: g-QAS_ASCS
   #      fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    <b>Started anftstsapcl2</b>
   #      nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      <b>Started anftstsapcl2</b>
   #      vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       <b>Started anftstsapcl2</b>
   #  Resource Group: g-QAS_ERS
   #      fs_QAS_ERS (ocf::heartbeat:Filesystem):    <b>Started anftstsapcl2</b>
   #      nc_QAS_ERS (ocf::heartbeat:azure-lb):      <b>Started anftstsapcl2</b>
   #      vip_QAS_ERS  (ocf::heartbeat:IPaddr2):     <b>Started anftstsapcl2</b>
   </code></pre>

4. **[2]** Instale SAP NetWeaver ERS

   Instale SAP NetWeaver ERS como raíz en el segundo nodo mediante un nombre de host virtual que se asigna a la dirección IP de la configuración de front-end del equilibrador de carga para ERS (por ejemplo, <b>anftstsapers</b>, <b>10.1.1.21</b>) y el número de instancia que usó para el sondeo del equilibrador de carga (por ejemplo, <b>01</b>).

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst. Puede usar el parámetro SAPINST_USE_HOSTNAME para instalar SAP con el nombre de host virtual.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b> SAPINST_USE_HOSTNAME=<b>virtual_hostname</b>
   </code></pre>

   > [!NOTE]
   > Use SWPM SP 20 PL 05 o superior. Las versiones inferiores no establecen correctamente los permisos y se producirá un error de instalación.

   Si se produce un error en la instalación para crear una subcarpeta en /usr/sap/**QAS**/ERS**01**, pruebe a establecer el propietario y el grupo de la carpeta ERS**01** e inténtelo de nuevo.

   <pre><code>
   chown qasadm /usr/sap/<b>QAS</b>/ERS<b>01</b>
   chgrp sapsys /usr/sap/<b>QAS</b>/ERS<b>01</b>
   </code></pre>


5. **[1]** Adapte los perfiles de instancias ASCS/SCS y ERS
 
   * Perfil ASCS/SCS

   <pre><code>
   sudo vi /sapmnt/<b>QAS</b>/profile/<b>QAS</b>_<b>ASCS00</b>_<b>anftstsapvh</b>
   
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

   <pre><code>
   sudo vi /sapmnt/<b>QAS</b>/profile/<b>QAS</b>_ERS<b>01</b>_<b>anftstsapers</b>
   
   # Change the restart command to a start command
   #Restart_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   Start_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   
   # Add the following lines
   service/halib = $(DIR_CT_RUN)/saphascriptco.so
   service/halib_cluster_connector = /usr/bin/sap_suse_cluster_connector
   
   # remove Autostart from ERS profile
   # Autostart = 1
   </code></pre>

6. **[A]** Configure la conexión persistente

   La comunicación entre el servidor de aplicaciones de SAP NetWeaver y ASCS/SCS se enruta a través de un equilibrador de carga de software. El equilibrador de carga desconecta las conexiones inactivas después de un tiempo de expiración que se puede configurar. Para evitar esto, debe establecer un parámetro en el perfil de SAP NetWeaver ASCS/SCS y cambiar la configuración del sistema Linux. Para más información, lea la [nota de SAP 1410736][1410736].

   <pre><code>
   # Change the Linux system configuration
   sudo sysctl net.ipv4.tcp_keepalive_time=120
   </code></pre>

7. **[A]** Configure los usuarios de SAP después de la instalación

   <pre><code>
   # Add sidadm to the haclient group
   sudo usermod -aG haclient <b>qas</b>adm
   </code></pre>

8. **[1]** Agregue los servicios SAP de ASCS y ERS al archivo `sapservice`

   Agregue la entrada del servicio ASCS al segundo nodo y copie la entrada del servicio ERS al primer nodo.

   <pre><code>
   cat /usr/sap/sapservices | grep ASCS<b>00</b> | sudo ssh <b>anftstsapcl2</b> "cat >>/usr/sap/sapservices"
   sudo ssh <b>anftstsapcl2</b> "cat /usr/sap/sapservices" | grep ERS<b>01</b> | sudo tee -a /usr/sap/sapservices
   </code></pre>

9. **[1]** Cree los recursos de clúster de SAP

Si usa la arquitectura de servidor 1 de puesta en cola (ENSA1), defina los recursos tal como se indica:

   <pre><code>sudo crm configure property maintenance-mode="true"
   
   sudo crm configure primitive rsc_sap_<b>QAS</b>_ASCS<b>00</b> SAPInstance \
    operations \$id=rsc_sap_<b>QAS</b>_ASCS<b>00</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>QAS</b>_ASCS<b>00</b>_<b>anftstsapvh</b> START_PROFILE="/sapmnt/<b>QAS</b>/profile/<b>QAS</b>_ASCS<b>00</b>_<b>anftstsapvh</b>" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 failure-timeout=60 migration-threshold=1 priority=10
   
   sudo crm configure primitive rsc_sap_<b>QAS</b>_ERS<b>01</b> SAPInstance \
    operations \$id=rsc_sap_<b>QAS</b>_ERS<b>01</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>QAS</b>_ERS<b>01</b>_<b>anftstsapers</b> START_PROFILE="/sapmnt/<b>QAS</b>/profile/<b>QAS</b>_ERS<b>01</b>_<b>anftstsapers</b>" AUTOMATIC_RECOVER=false IS_ERS=true \
    meta priority=1000
   
   sudo crm configure modgroup g-<b>QAS</b>_ASCS add rsc_sap_<b>QAS</b>_ASCS<b>00</b>
   sudo crm configure modgroup g-<b>QAS</b>_ERS add rsc_sap_<b>QAS</b>_ERS<b>01</b>
   
   sudo crm configure colocation col_sap_<b>QAS</b>_no_both -5000: g-<b>QAS</b>_ERS g-<b>QAS</b>_ASCS
   sudo crm configure location loc_sap_<b>QAS</b>_failover_to_ers rsc_sap_<b>QAS</b>_ASCS<b>00</b> rule 2000: runs_ers_<b>QAS</b> eq 1
   sudo crm configure order ord_sap_<b>QAS</b>_first_start_ascs Optional: rsc_sap_<b>QAS</b>_ASCS<b>00</b>:start rsc_sap_<b>QAS</b>_ERS<b>01</b>:stop symmetrical=false
   
   sudo crm node online <b>anftstsapcl1</b>
   sudo crm configure property maintenance-mode="false"
   </code></pre>

   SAP introdujo una opción de compatibilidad con el servidor 2 de puesta en cola, incluida la replicación, a partir de la versión de SAP NW 7.52. A partir de la plataforma ABAP (versión 1809), el servidor 2 de puesta en cola está instalado de forma predeterminada. Consulte la nota de SAP [2630416](https://launchpad.support.sap.com/#/notes/2630416) para consultar la compatibilidad con el servidor 2 de puesta en cola.
Si usa la arquitectura de servidor 2 de puesta en cola ([ENSA2](https://help.sap.com/viewer/cff8531bc1d9416d91bb6781e628d4e0/1709%20001/en-US/6d655c383abf4c129b0e5c8683e7ecd8.html)), defina los recursos tal como se indica:

   <pre><code>sudo crm configure property maintenance-mode="true"
   
   sudo crm configure primitive rsc_sap_<b>QAS</b>_ASCS<b>00</b> SAPInstance \
    operations \$id=rsc_sap_<b>QAS</b>_ASCS<b>00</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>QAS</b>_ASCS<b>00</b>_<b>anftstsapvh</b> START_PROFILE="/sapmnt/<b>QAS</b>/profile/<b>QAS</b>_ASCS<b>00</b>_<b>anftstsapvh</b>" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000
   
   sudo crm configure primitive rsc_sap_<b>QAS</b>_ERS<b>01</b> SAPInstance \
    operations \$id=rsc_sap_<b>QAS</b>_ERS<b>01</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>QAS</b>_ERS<b>01</b>_<b>anftstsapers</b> START_PROFILE="/sapmnt/<b>QAS</b>/profile/<b>QAS</b>_ERS<b>01</b>_<b>anftstsapers</b>" AUTOMATIC_RECOVER=false IS_ERS=true
   
   sudo crm configure modgroup g-<b>QAS</b>_ASCS add rsc_sap_<b>QAS</b>_ASCS<b>00</b>
   sudo crm configure modgroup g-<b>QAS</b>_ERS add rsc_sap_<b>QAS</b>_ERS<b>01</b>
   
   sudo crm configure colocation col_sap_<b>QAS</b>_no_both -5000: g-<b>QAS</b>_ERS g-<b>QAS</b>_ASCS
   sudo crm configure order ord_sap_<b>QAS</b>_first_start_ascs Optional: rsc_sap_<b>QAS</b>_ASCS<b>00</b>:start rsc_sap_<b>QAS</b>_ERS<b>01</b>:stop symmetrical=false
   
   sudo crm node online <b>anftstsapcl1</b>
   sudo crm configure property maintenance-mode="false"
   </code></pre>

   Si está actualizando desde una versión anterior y va a cambiar al servidor 2 de puesta en cola, consulte la nota de SAP [2641019](https://launchpad.support.sap.com/#/notes/2641019). 

   Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.

   <pre><code>sudo crm_mon -r
   # Full list of resources:
   #
   # stonith-sbd     (stonith:external/sbd): <b>Started anftstsapcl2</b>
   #  Resource Group: g-QAS_ASCS
   #      fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    <b>Started anftstsapcl1</b>
   #      nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      <b>Started anftstsapcl1</b>
   #      vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       <b>Started anftstsapcl1</b>
   #      rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   <b>Started anftstsapcl1</b>
   #  Resource Group: g-QAS_ERS
   #      fs_QAS_ERS (ocf::heartbeat:Filesystem):    <b>Started anftstsapcl2</b>
   #      nc_QAS_ERS (ocf::heartbeat:azure-lb):      <b>Started anftstsapcl2</b>
   #      vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       <b>Started anftstsapcl2</b>
   #      rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   <b>Started anftstsapcl2</b>
   </code></pre>

## <a name="sap-netweaver-application-server-preparation"></a><a name="2d6008b0-685d-426c-b59e-6cd281fd45d7"></a>Preparación del servidor de aplicaciones de SAP NetWeaver 

Algunas bases de datos requieren que la instalación de la instancia de base de datos se ejecute en un servidor de aplicaciones. Prepare las máquinas virtuales del servidor de aplicaciones para poder usarlas en estos casos.

En los pasos siguientes se supone que instala el servidor de aplicaciones en un servidor distinto de los servidores ASCS/SCS y HANA. De lo contrario, no se necesitan algunos de los pasos que aparecen a continuación (como configurar la resolución de nombres de host).

Los elementos siguientes tienen el prefijo **[A]** (aplicable a PAS y AAS), **[P]** (solo aplicable a PAS) o **[S]** (solo aplicable a AAS).


1. **[A]** Configure el sistema operativo

   Reduzca el tamaño de la caché de datos incorrectos. Para más información, consulte [Low write performance on SLES 11/12 servers with large RAM](https://www.suse.com/support/kb/doc/?id=7010287) (Bajo rendimiento de escritura en servidores SLES 11/12 con RAM grande).

   <pre><code>
   sudo vi /etc/sysctl.conf
   # Change/set the following settings
   vm.dirty_bytes = 629145600
   vm.dirty_background_bytes = 314572800
   </code></pre>

1. **[A]** Configure la resolución nombres de host

   Puede usar un servidor DNS o modificar /etc/hosts en todos los nodos. En este ejemplo se muestra cómo utilizar el archivo /etc/hosts.
   Reemplace la dirección IP y el nombre de host en los siguientes comandos

   ```bash
   sudo vi /etc/hosts
   ```

   Inserte las siguientes líneas en /etc/hosts. Cambie la dirección IP y el nombre de host para que coincida con su entorno

   <pre><code>
   # IP address of the load balancer frontend configuration for SAP NetWeaver ASCS/SCS
   <b>10.1.1.20 anftstsapvh</b>
   # IP address of the load balancer frontend configuration for SAP NetWeaver ERS
   <b>10.1.1.21 anftstsapers</b>
   # IP address of all application servers
   <b>10.1.1.15 anftstsapa01</b>
   <b>10.1.1.16 anftstsapa02</b>
   </code></pre>

1. **[A]** Cree el directorio sapmnt.

   <pre><code>
   sudo mkdir -p /sapmnt/<b>QAS</b>
   sudo mkdir -p /usr/sap/trans

   sudo chattr +i /sapmnt/<b>QAS</b>
   sudo chattr +i /usr/sap/trans
   </code></pre>

1. **[P]** Cree el directorio PAS.

   <pre><code>
   sudo mkdir -p /usr/sap/<b>QAS</b>/D<b>02</b>
   sudo chattr +i /usr/sap/<b>QAS</b>/D<b>02</b>
   </code></pre>

1. **[S]** Cree el directorio AAS.

   <pre><code>
   sudo mkdir -p /usr/sap/<b>QAS</b>/D<b>03</b>
   sudo chattr +i /usr/sap/<b>QAS</b>/D<b>03</b>
   </code></pre>

1. **[P]** Configure `autofs` en PAS

   <pre><code>sudo vi /etc/auto.master
   
   # Add the following line to the file, save and exit
   /- /etc/auto.direct
   </code></pre>

   Si usa NFSv3, cree un archivo con:

   <pre><code>
   sudo vi /etc/auto.direct
   # Add the following lines to the file, save and exit
   /sapmnt/<b>QAS</b> -nfsvers=3,nobind <b>10.1.0.4</b>:/usrsap<b>qas</b>/sapmnt<b>QAS</b>
   /usr/sap/trans -nfsvers=3,nobind <b>10.1.0.4</b>:/trans
   /usr/sap/<b>QAS</b>/D<b>02</b> -nfsvers=3,nobind <b>10.1.0.4</b>:/usrsap<b>qas</b>/usrsap<b>QAS</b>pas
   </code></pre>

   Si usa NFSv4.1, cree un archivo con:

   <pre><code>
   sudo vi /etc/auto.direct
   # Add the following lines to the file, save and exit
   /sapmnt/<b>QAS</b> -nfsvers=4.1,nobind,sec=sys <b>10.1.0.4</b>:/usrsap<b>qas</b>/sapmnt<b>QAS</b>
   /usr/sap/trans -nfsvers=4.1,nobind,sec=sys <b>10.1.0.4</b>:/trans
   /usr/sap/<b>QAS</b>/D<b>02</b> -nfsvers=4.1,nobind,sec=sys <b>10.1.0.4</b>:/usrsap<b>qas</b>/usrsap<b>QAS</b>pas
   </code></pre>

   Reinicie `autofs` para montar los recursos compartidos nuevos.

   <pre><code>
   sudo systemctl enable autofs
   sudo service autofs restart
   </code></pre>

1. **[P]** Configure `autofs` en AAS

   <pre><code>sudo vi /etc/auto.master
   
   # Add the following line to the file, save and exit
   /- /etc/auto.direct
   </code></pre>

   Si usa NFSv3, cree un archivo con:

   <pre><code>
   sudo vi /etc/auto.direct
   # Add the following lines to the file, save and exit
   /sapmnt/<b>QAS</b> -nfsvers=3,nobind <b>10.1.0.4</b>:/usrsap<b>qas</b>/sapmnt<b>QAS</b>
   /usr/sap/trans -nfsvers=3,nobind <b>10.1.0.4</b>:/trans
   /usr/sap/<b>QAS</b>/D<b>03</b> -nfsvers=3,nobind <b>10.1.0.4</b>:/usrsap<b>qas</b>/usrsap<b>QAS</b>aas
   </code></pre>

   Si usa NFSv4.1, cree un archivo con:

   <pre><code>
   sudo vi /etc/auto.direct
   # Add the following lines to the file, save and exit
   /sapmnt/<b>QAS</b> -nfsvers=4.1,nobind,sec=sys <b>10.1.0.4</b>:/usrsap<b>qas</b>/sapmnt<b>QAS</b>
   /usr/sap/trans -nfsvers=4.1,nobind,sec=sys <b>10.1.0.4</b>:/trans
   /usr/sap/<b>QAS</b>/D<b>03</b> -nfsvers=4.1,nobind,sec=sys <b>10.1.0.4</b>:/usrsap<b>qas</b>/usrsap<b>QAS</b>aas
   </code></pre>

   Reinicie `autofs` para montar los recursos compartidos nuevos.

   <pre><code>
   sudo systemctl enable autofs
   sudo service autofs restart
   </code></pre>

1. **[A]** Configure el archivo de intercambio

   <pre><code>
   sudo vi /etc/waagent.conf
   
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

* Ejecute la instalación de la instancia de base de datos de SAP.

   Instale la instancia de base de datos de SAP NetWeaver como raíz con un nombre de host virtual que se asigna a la dirección IP de la configuración de front-end del equilibrador de carga para la base de datos.

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b>
   </code></pre>

## <a name="sap-netweaver-application-server-installation"></a>Instalación del servidor de aplicaciones de SAP NetWeaver

Siga estos pasos para instalar un servidor de aplicaciones de SAP.

1. **[A]** Prepare el servidor de aplicaciones. Siga los pasos descritos en el capítulo [Preparación del servidor de aplicaciones de SAP NetWeaver](high-availability-guide-suse-netapp-files.md#2d6008b0-685d-426c-b59e-6cd281fd45d7) anterior para preparar el servidor de aplicaciones.

2. **[A]** Instale el servidor de aplicaciones de SAP NetWeaver. Instale un servidor de aplicaciones de SAP NetWeaver principal o adicional.

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b>
   </code></pre>

3. **[A]** Actualice el almacenamiento seguro de SAP HANA.

   Actualice el almacenamiento seguro de SAP HANA que apunte al nombre virtual de la configuración de la replicación del sistema SAP HANA.

   Ejecute el siguiente comando para mostrar las entradas:
   <pre><code>
   hdbuserstore List
   </code></pre>

   Se deberían mostrar todas las entradas y deberían ser parecidas a estas:
   <pre><code>
   DATA FILE       : /home/qasadm/.hdb/anftstsapa01/SSFS_HDB.DAT
   KEY FILE        : /home/qasadm/.hdb/anftstsapa01/SSFS_HDB.KEY
   
   KEY DEFAULT
     ENV : 10.1.1.5:<b>30313</b>
     USER: <b>SAPABAP1</b>
     DATABASE: <b>QAS</b>
   </code></pre>

   El resultado muestra que la dirección IP de la entrada predeterminada apunta a la máquina virtual y no a la dirección IP del equilibrador de carga. Esta entrada debe modificarse para que apunte al nombre de host virtual del equilibrador de carga. Asegúrese de usar el mismo puerto (**30313** en la salida anterior) y el mismo nombre de base de datos (**QAS** en la salida anterior).

   <pre><code>
   su - <b>qas</b>adm
   hdbuserstore SET DEFAULT <b>qasdb:30313@QAS</b> <b>SAPABAP1</b> <b>&lt;password of ABAP schema&gt;</b>
   </code></pre>

## <a name="test-the-cluster-setup"></a>Prueba de la configuración del clúster

Las siguientes pruebas son una copia de los casos de prueba de las [guías de procedimientos recomendados de SUSE][suse-ha-guide]. Para su comodidad, se han copiado en este artículo. Además, lea siempre las guías de procedimientos recomendados y realice todas las pruebas adicionales que puedan haberse agregado.

1. Prueba de HAGetFailoverConfig, HACheckConfig y HACheckFailoverConfig

   Ejecute los siguientes comandos como \<sapsid>adm en el nodo donde se ejecuta actualmente la instancia de ASCS. Si los comandos producen un error de memoria insuficiente, los guiones en el nombre de host pueden ser la causa. Se trata de un problema conocido que SUSE corregirá en el paquete sap-suse-cluster-connector.

   <pre><code>
   anftstsapcl1:qasadm 52> sapcontrol -nr 00 -function HAGetFailoverConfig
   07.03.2019 20:08:59
   HAGetFailoverConfig
   OK
   HAActive: TRUE
   HAProductVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP3
   HASAPInterfaceVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP3 (sap_suse_cluster_connector 3.1.0)
   HADocumentation: https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
   HAActiveNode: anftstsapcl1
   HANodes: anftstsapcl1, anftstsapcl2

   anftstsapcl1:qasadm 54> sapcontrol -nr 00 -function HACheckConfig
   07.03.2019 23:28:29
   HACheckConfig
   OK
   state, category, description, comment
   SUCCESS, SAP CONFIGURATION, Redundant ABAP instance configuration, 2 ABAP instances detected
   SUCCESS, SAP CONFIGURATION, Redundant Java instance configuration, 0 Java instances detected
   SUCCESS, SAP CONFIGURATION, Enqueue separation, All Enqueue server separated from application server
   SUCCESS, SAP CONFIGURATION, MessageServer separation, All MessageServer separated from application server
   SUCCESS, SAP CONFIGURATION, ABAP instances on multiple hosts, ABAP instances on multiple hosts detected
   SUCCESS, SAP CONFIGURATION, Redundant ABAP SPOOL service configuration, 2 ABAP instances with SPOOL service detected
   SUCCESS, SAP STATE, Redundant ABAP SPOOL service state, 2 ABAP instances with active SPOOL service detected
   SUCCESS, SAP STATE, ABAP instances with ABAP SPOOL service on multiple hosts, ABAP instances with active ABAP SPOOL service on multiple hosts detected
   SUCCESS, SAP CONFIGURATION, Redundant ABAP BATCH service configuration, 2 ABAP instances with BATCH service detected
   SUCCESS, SAP STATE, Redundant ABAP BATCH service state, 2 ABAP instances with active BATCH service detected
   SUCCESS, SAP STATE, ABAP instances with ABAP BATCH service on multiple hosts, ABAP instances with active ABAP BATCH service on multiple hosts detected
   SUCCESS, SAP CONFIGURATION, Redundant ABAP DIALOG service configuration, 2 ABAP instances with DIALOG service detected
   SUCCESS, SAP STATE, Redundant ABAP DIALOG service state, 2 ABAP instances with active DIALOG service detected
   SUCCESS, SAP STATE, ABAP instances with ABAP DIALOG service on multiple hosts, ABAP instances with active ABAP DIALOG service on multiple hosts detected
   SUCCESS, SAP CONFIGURATION, Redundant ABAP UPDATE service configuration, 2 ABAP instances with UPDATE service detected
   SUCCESS, SAP STATE, Redundant ABAP UPDATE service state, 2 ABAP instances with active UPDATE service detected
   SUCCESS, SAP STATE, ABAP instances with ABAP UPDATE service on multiple hosts, ABAP instances with active ABAP UPDATE service on multiple hosts detected
   SUCCESS, SAP STATE, SCS instance running, SCS instance status ok
   SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version (anftstsapvh_QAS_00), SAPInstance includes is-ers patch
   SUCCESS, SAP CONFIGURATION, Enqueue replication (anftstsapvh_QAS_00), Enqueue replication enabled
   SUCCESS, SAP STATE, Enqueue replication state (anftstsapvh_QAS_00), Enqueue replication active
   
   anftstsapcl1:qasadm 55> sapcontrol -nr 00 -function HACheckFailoverConfig
   07.03.2019 23:30:48
   HACheckFailoverConfig
   OK
   state, category, description, comment
   SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version, SAPInstance includes is-ers patch
   </code></pre>

2. Migración manual de la instancia de ASCS

   Estado del recurso antes de iniciar la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rscsap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Starting anftstsapcl1
   </code></pre>

   Ejecute los siguientes comandos como raíz para migrar la instancia de ASCS.

   <pre><code>
   anftstsapcl1:~ # crm resource migrate rsc_sap_QAS_ASCS00 force
   INFO: Move constraint created for rsc_sap_QAS_ASCS00
   
   anftstsapcl1:~ # crm resource unmigrate rsc_sap_QAS_ASCS00
   INFO: Removed migration constraints for rsc_sap_QAS_ASCS00
   
   # Remove failed actions for the ERS that occurred as part of the migration
   anftstsapcl1:~ # crm resource cleanup rsc_sap_QAS_ERS01
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   </code></pre>

3. Prueba de HAFailoverToNode

   Estado del recurso antes de iniciar la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   </code></pre>

   Ejecute los siguientes comandos como \<sapsid>administrador para migrar la instancia de ASCS.

   <pre><code>
   anftstsapcl1:qasadm 53> sapcontrol -nr 00 -host anftstsapvh -user <b>qas</b>adm &lt;password&gt; -function HAFailoverToNode ""
   
   # run as root
   # Remove failed actions for the ERS that occurred as part of the migration
   anftstsapcl1:~ # crm resource cleanup rsc_sap_QAS_ERS01
   # Remove migration constraints
   anftstsapcl1:~ # crm resource clear rsc_sap_QAS_ASCS00
   #INFO: Removed migration constraints for rsc_sap_QAS_ASCS00
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

4. Simulación de bloqueo de nodo 

   Estado del recurso antes de iniciar la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

   Ejecute el siguiente comando como raíz en el nodo donde se ejecuta la instancia de ASCS

   <pre><code>anftstsapcl2:~ # echo b > /proc/sysrq-trigger
   </code></pre>

   Si usa SBD, Pacemaker no se iniciará automáticamente en el nodo terminado. El estado después de que el nodo se inicia de nuevo debe parecerse a este.

   <pre><code>Online:
   Online: [ anftstsapcl1 ]
   OFFLINE: [ anftstsapcl2 ]

   Full list of resources:

    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1

   Failed Actions:
   * rsc_sap_QAS_ERS01_monitor_11000 on anftstsapcl1 'not running' (7): call=166, status=complete, exitreason='',
    last-rc-change='Fri Mar  8 18:26:10 2019', queued=0ms, exec=0ms
   </code></pre>

   Use los siguientes comandos para iniciar Pacemaker en el nodo terminado, limpiar los mensajes de SBD y limpiar los recursos con error.

   <pre><code>
   # run as root
   # list the SBD device(s)
   anftstsapcl2:~ # cat /etc/sysconfig/sbd | grep SBD_DEVICE=
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405b730e31e7d5a4516a2a697dcf;/dev/disk/by-id/scsi-36001405f69d7ed91ef54461a442c676e;/dev/disk/by-id/scsi-360014058e5f335f2567488882f3a2c3a"

   anftstsapcl2:~ # sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e11 -d /dev/disk/by-id/scsi-36001405f69d7ed91ef54461a442c676e -d /dev/disk/by-id/scsi-360014058e5f335f2567488882f3a2c3a message anftstsapcl2 clear

   anftstsapcl2:~ # systemctl start pacemaker
   anftstsapcl2:~ # crm resource cleanup rsc_sap_QAS_ASCS00
   anftstsapcl2:~ # crm resource cleanup rsc_sap_QAS_ERS01
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>
   Full list of resources:
   
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   </code></pre>

5. Prueba del reinicio manual de la instancia de ASCS

   Estado del recurso antes de iniciar la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

   Cree un bloqueo de puesta en cola; por ejemplo, edite un usuario en la transacción su01. Ejecute los comandos siguientes como <sapsid\>adm en el nodo donde se ejecuta la instancia de ASCS. Los comandos detendrán la instancia de ASCS y la volverán a iniciar. Si usa la arquitectura deL servidor 1 de puesta en cola, es posible que el bloqueo de puesta en cola se pierda en esta prueba. Si usa la arquitectura del servidor 2 de puesta en cola, se conservará la puesta en cola. 

   <pre><code>anftstsapcl2:qasadm 51> sapcontrol -nr 00 -function StopWait 600 2
   </code></pre>

   Ahora, la instancia de ASCS se deshabilitará en Pacemaker.

   <pre><code>  rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Stopped (disabled)
   </code></pre>

   Vuelva a iniciar la instancia de ASCS en el mismo nodo.

   <pre><code>anftstsapcl2:qasadm 52> sapcontrol -nr 00 -function StartWait 600 2
   </code></pre>

   El bloqueo de puesta en cola de la transacción su01 se perderá, si se usa la arquitectura de replicación 1 del servidor de colas, y el back-end se habrá restablecido. Estado del recurso después de la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

6. Terminación del proceso del servidor de mensajes

   Estado del recurso antes de iniciar la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

   Ejecute los siguientes comandos como raíz para identificar el proceso del servidor de mensajes y terminarlo.

   <pre><code>anftstsapcl2:~ # pgrep ms.sapQAS | xargs kill -9
   </code></pre>

   Si solo termina el servidor de mensajes una vez, se reiniciará mediante `sapstart`. Si lo termina con bastante frecuencia, Pacemaker acabará moviendo la instancia de ASCS al otro nodo. Ejecute los siguientes comandos como raíz para limpiar el estado del recurso de la instancia de ASCS y ERS después de la prueba.

   <pre><code>
   anftstsapcl2:~ # crm resource cleanup rsc_sap_QAS_ASCS00
   anftstsapcl2:~ # crm resource cleanup rsc_sap_QAS_ERS01
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   </code></pre>

7. Terminación del proceso del servidor de puesta en cola

   Estado del recurso antes de iniciar la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   </code></pre>

   Ejecute los siguientes comandos como raíz en el nodo donde se ejecuta la instancia de ASCS para terminar el servidor de puesta en cola.

   <pre><code>anftstsapcl1:~ # pgrep en.sapQAS | xargs kill -9
   </code></pre>

   La instancia de ASCS conmutará inmediatamente por error al otro nodo. La instancia de ERS también conmutará por error después de iniciarse la instancia de ASCS. Ejecute los siguientes comandos como raíz para limpiar el estado del recurso de la instancia de ASCS y ERS después de la prueba.

   <pre><code>
   anftstsapcl1:~ # crm resource cleanup rsc_sap_QAS_ASCS00
   anftstsapcl1:~ # crm resource cleanup rsc_sap_QAS_ERS01
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

8. Terminación del proceso del servidor de replicación

   Estado del recurso antes de iniciar la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

   Ejecute el siguiente comando como raíz en el nodo donde se ejecuta la instancia de ERS para terminar el proceso del servidor de replicación de puesta en cola.

   <pre><code>anftstsapcl1:~ # pgrep er.sapQAS | xargs kill -9
   </code></pre>

   Si solo ejecuta el comando una vez, `sapstart` reiniciará el proceso. Si lo ejecuta con bastante frecuencia, `sapstart` no reiniciará el proceso y el recurso estará en estado detenido. Ejecute los siguientes comandos como raíz para limpiar el estado del recurso de la instancia de ERS después de la prueba.

   <pre><code>anftstsapcl1:~ # crm resource cleanup rsc_sap_QAS_ERS01
   </code></pre>

   Estado del recurso después de la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

9. Terminación del proceso sapstartsrv de puesta en cola

   Estado del recurso antes de iniciar la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

   Ejecute los siguientes comandos como raíz en el nodo donde se ejecuta ASCS.

   <pre><code>
   anftstsapcl2:~ # pgrep -fl ASCS00.*sapstartsrv
   #67625 sapstartsrv
   
   anftstsapcl2:~ # kill -9 67625
   </code></pre>

   El proceso sapstartsrv siempre se debe reiniciar con el agente de recursos de Pacemaker. Estado del recurso después de la prueba:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

## <a name="next-steps"></a>Pasos siguientes

* [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications: guía de varios SID](./high-availability-guide-suse-multi-sid.md)
* [Planeamiento e implementación de Azure Virtual Machines para SAP][planning-guide]
* [Implementación de Azure Virtual Machines para SAP][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP][dbms-guide]
* Para más información sobre cómo establecer la alta disponibilidad y planear la recuperación ante desastres de SAP HANA en Azure Virtual Machines, consulte [Alta disponibilidad de SAP HANA en las máquinas virtuales de Azure][sap-hana-ha].
