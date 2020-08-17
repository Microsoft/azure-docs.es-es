---
title: Alta disponibilidad de máquinas virtuales de Azure para SAP NW en RHEL con Azure NetApp Files | Microsoft Docs
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
ms.openlocfilehash: a8503f701a36d7197f85f2c86e37272ccf1cab48
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87760726"
---
# <a name="azure-virtual-machines-high-availability-for-sap-netweaver-on-red-hat-enterprise-linux-with-azure-netapp-files-for-sap-applications"></a>Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver en Red Hat Enterprise Linux con Azure NetApp Files para aplicaciones SAP

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

En este artículo se describe cómo implementar y configurar las máquinas virtuales, y cómo instalar el marco del clúster y un sistema SAP NetWeaver 7.50 de alta disponibilidad, mediante [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md).
En las configuraciones, comandos de instalación, etc. de ejemplo, La instancia de ASCS tiene el número 00, la de ERS el 01, la de la aplicación principal (PAS) el 02 y la de la aplicación (AAS) el 03. Se usan los controles de calidad de los identificadores del sistema SAP. 

El nivel de la base de datos no se trata en detalle en este artículo.  

Lea primero las notas y los documentos de SAP siguientes:

* [Documentación sobre Azure NetApp Files][anf-azure-doc] 
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
* [Implementación de Azure Virtual Machines para SAP en Linux][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP en Linux][dbms-guide]
* [SAP Netweaver en el clúster de Pacemaker](https://access.redhat.com/articles/3150081)
* Documentación general de RHEL
  * [Introducción al complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [Administración del complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [Referencia del complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
  * [Configuración de ASCS/ERS para SAP Netweaver con recursos independientes en RHEL 7.5](https://access.redhat.com/articles/3569681)
  * [Configuración de SAP S/4HANA ASCS/ERS con el servidor 2 de puesta en cola independiente (ENSA2) en Pacemaker en RHEL ](https://access.redhat.com/articles/3974941)
* Documentación de RHEL específica para Azure:
  * [Directivas de compatibilidad para clústeres de alta disponibilidad RHEL: instancias de Microsoft Azure Virtual Machines como miembros del clúster](https://access.redhat.com/articles/3131341)
  * [Instalación y configuración de un clúster de alta disponibilidad de Red Hat Enterprise Linux 7.4 (y versiones posteriores) en Microsoft Azure](https://access.redhat.com/articles/3252491)
* [Aplicaciones de NetApp SAP en Microsoft Azure que usan Azure NetApp Files][anf-sap-applications-azure]

## <a name="overview"></a>Información general

La alta disponibilidad de los servicios centrales de SAP Netweaver requiere almacenamiento compartido.
Para lograrlo en Red Hat Linux hasta ahora era necesario crear el clúster independiente GlusterFS de alta disponibilidad. 

Ahora ya es posible lograr alta disponibilidad en SAP NetWeaver mediante el uso de almacenamiento compartido implementado en Azure NetApp Files. Mediante el uso de Azure NetApp Files para el almacenamiento compartido se elimina la necesidad del [clúster GlusterFS](./high-availability-guide-rhel-glusterfs.md) adicional. Pacemaker sigue siendo necesario para lograr una alta disponibilidad de los servicios centrales de SAP NetWeaver (ASCS/SCS).

![Información general sobre la alta disponibilidad de SAP NetWeaver](./media/high-availability-guide-rhel/high-availability-guide-rhel-anf.png)

SAP NetWeaver ASCS, SAP NetWeaver SCS, SAP NetWeaver ERS y la base de datos SAP HANA usan direcciones IP virtuales y el nombre de host virtual. En Azure, se requiere un equilibrador de carga para usar una dirección IP virtual. Se recomienda usar [Standard Load Balancer](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md). En la lista siguiente se muestra la configuración del equilibrador de carga con distintas direcciones IP de front-end para (A)SCS y ERS.

### <a name="ascs"></a>(A)SCS

* Configuración de front-end
  * Dirección IP 192.168.14.9
* Puerto de sondeo
  * Puerto 620<strong>&lt;nr&gt;</strong>
* Reglas de equilibrio de carga.
  * Si usa Standard Load Balancer, seleccione **Puertos HA**
  * 32<strong>&lt;nr&gt;</strong> TCP
  * 36<strong>&lt;nr&gt;</strong> TCP
  * 39<strong>&lt;nr&gt;</strong> TCP
  * 81<strong>&lt;nr&gt;</strong> TCP
  * 5<strong>&lt;nr&gt;</strong>13 TCP
  * 5<strong>&lt;nr&gt;</strong>14 TCP
  * 5<strong>&lt;nr&gt;</strong>16 TCP

### <a name="ers"></a>ERS

* Configuración de front-end
  * Dirección IP 192.168.14.10
* Puerto de sondeo
  * Puerto 621<strong>&lt;nr&gt;</strong>
* Reglas de equilibrio de carga.
  * Si usa Standard Load Balancer, seleccione **Puertos HA**
  * 32<strong>&lt;nr&gt;</strong> TCP
  * 33<strong>&lt;nr&gt;</strong> TCP
  * 5<strong>&lt;nr&gt;</strong>13 TCP
  * 5<strong>&lt;nr&gt;</strong>14 TCP
  * 5<strong>&lt;nr&gt;</strong>16 TCP

* Configuración de back-end
  * Se conecta a interfaces de red principales de todas las máquinas que deben ser parte del clúster (A)SCS/ERS

## <a name="setting-up-the-azure-netapp-files-infrastructure"></a>Configuración de la infraestructura de Azure NetApp Files 

SAP NetWeaver requiere un almacenamiento compartido para el directorio de transporte y perfil.  Antes de continuar con la configuración de la infraestructura de Azure NetApp Files, familiarícese con la [documentación correspondiente][anf-azure-doc]. Compruebe si la región de Azure seleccionada ofrece Azure NetApp Files. El siguiente vínculo muestra la disponibilidad de Azure NetApp Files por región de Azure: [Disponibilidad de Azure NetApp Files por región de Azure][anf-avail-matrix].

Azure NetApp Files está disponible en varias [regiones de Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp). Antes de implementar Azure NetApp Files, solicite la incorporación a este siguiendo las [instrucciones para registrarse en Azure NetApp Files][anf-register]. 

### <a name="deploy-azure-netapp-files-resources"></a>Implementación de recursos de Azure NetApp Files  

En estos pasos se supone que ya ha implementado [Azure Virtual Network](../../../virtual-network/virtual-networks-overview.md). Los recursos de Azure NetApp Files y las máquinas virtuales en las que esos recursos se montarán se deben implementar en la misma red virtual de Azure o en redes virtuales emparejadas.  

1. Si aún no lo ha hecho, solicite la [incorporación a Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).  
2. Cree la cuenta de NetApp en la región de Azure seleccionada mediante las [instrucciones para crear la cuenta de NetApp](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md).  
3. Configure el grupo de capacidad de Azure NetApp Files con las [instrucciones correspondientes](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md).  
La arquitectura de SAP Netweaver que se presenta en este artículo usa un único grupo de capacidad de Azure NetApp Files, SKU Premium. Se recomienda usar SKU Premium de Azure NetApp Files para cargas de trabajo de aplicaciones de SAP NetWeaver en Azure.  
4. Delegue una subred en Azure NetApp Files como se describe en las [instrucciones correspondientes](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).  

5. Implemente los volúmenes de Azure NetApp Files mediante las [instrucciones correspondientes](../../../azure-netapp-files/azure-netapp-files-create-volumes.md). Implemente los volúmenes en la [subred](/rest/api/virtualnetwork/subnets) de Azure NetApp Files designada. Las direcciones IP de los volúmenes de Azure NetApp se asignan automáticamente. Tenga en cuenta que los recursos de Azure NetApp Files y las máquinas virtuales de Azure deben estar en la misma red virtual de Azure o en redes virtuales emparejadas. En este ejemplo se usan dos volúmenes de Azure NetApp Files: sap<b>QAS</b> y transSAP. Las rutas de acceso de los archivos que se montan en los correspondientes puntos de montaje son /usrsap<b>qas</b>/sapmnt<b>QAS</b>, /usrsap<b>qas</b>/usrsap<b>QAS</b>sys, etc.  

   1. volumen sap<b>QAS</b> (nfs://192.168.24.5/usrsap<b>qas</b>/sapmnt<b>QAS</b>)
   2. volumen sap<b>QAS</b> (nfs://192.168.24.5/usrsap<b>qas</b>/usrsap<b>QAS</b>ascs)
   3. volumen sap<b>QAS</b> (nfs://192.168.24.5/usrsap<b>qas</b>/usrsap<b>QAS</b>sys)
   4. volumen sap<b>QAS</b> (nfs://192.168.24.5/usrsap<b>qas</b>/usrsap<b>QAS</b>ers)
   5. volumen transSAP (nfs://192.168.24.4/transSAP)
   6. volumen sap<b>QAS</b> (nfs://192.168.24.5/usrsap<b>qas</b>/usrsap<b>QAS</b>pas)
   7. volumen sap<b>QAS</b> (nfs://192.168.24.5/usrsap<b>qas</b>/usrsap<b>QAS</b>aas)
  
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

## <a name="setting-up-ascs"></a>Configuración de (A)SCS

En este ejemplo, los recursos se implementaron manualmente mediante [Azure Portal](https://portal.azure.com/#home).

### <a name="deploy-linux-manually-via-azure-portal"></a>Implementación manual de Linux mediante Azure Portal

Primero deberá crear los volúmenes de Azure NetApp Files. Implemente las VM. Después, creará un equilibrador de carga y usará las máquinas virtuales del grupo de servidores back-end.

1. Cree un equilibrador de carga (interno, estándar):  
   1. Creación de las direcciones IP de front-end
      1. Dirección IP 192.168.14.9 de ASCS
         1. Abra el equilibrador de carga, seleccione el grupo de direcciones IP de front-end y haga clic en Agregar
         1. Escriba el nombre del nuevo grupo de direcciones IP de front-end (por ejemplo, **frontend.QAS.ASCS**).
         1. Establezca Asignación en Estática y escriba la dirección IP (por ejemplo, **192.168.14.9**).
         1. Haga clic en Aceptar
      1. Dirección IP 192.168.14.10 para ASCS ERS
         * Repita los pasos anteriores a partir de "a" para crear una dirección IP para el ERS (por ejemplo, **192.168.14.10** y **frontend.QAS.ERS**).
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
      1. Reglas de equilibrio de carga para ASCS
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
      1. Dirección IP 192.168.14.9 de ASCS
         1. Abra el equilibrador de carga, seleccione el grupo de direcciones IP de front-end y haga clic en Agregar
         1. Escriba el nombre del nuevo grupo de direcciones IP de front-end (por ejemplo, **frontend.QAS.ASCS**).
         1. Establezca Asignación en Estática y escriba la dirección IP (por ejemplo, **192.168.14.9**).
         1. Haga clic en Aceptar
      1. Dirección IP 192.168.14.10 para ASCS ERS
         * Repita los pasos anteriores a partir de "a" para crear una dirección IP para el ERS (por ejemplo, **192.168.14.10** y **frontend.QAS.ERS**).
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

## <a name="disable-id-mapping-if-using-nfsv41"></a>Deshabilitar la asignación de identificadores (si se usa NFSv4.1)

Las instrucciones de esta sección solo se aplican si se usan volúmenes de Azure NetApp Files con el protocolo NFSv4.1. Realice la configuración en todas las máquinas virtuales donde se montarán los volúmenes NFSv4.1 de Azure NetApp Files.  

1. Compruebe la configuración del dominio NFS. Asegúrese de que el dominio esté configurado como dominio predeterminado de Azure NetApp Files, es decir, **`defaultv4iddomain.com`** y que la asignación se haya establecido en **nobody**.  

    > [!IMPORTANT]
    > Asegúrese de establecer el dominio NFS de `/etc/idmapd.conf` en la máquina virtual para que coincida con la configuración de dominio predeterminada en Azure NetApp Files: **`defaultv4iddomain.com`** . Si hay alguna discrepancia entre la configuración de dominio del cliente NFS (es decir, la máquina virtual) y el servidor NFS (es decir la configuración de Azure NetApp), los permisos de archivos en volúmenes de Azure NetApp que estén montados en las máquinas virtuales se mostrarán como `nobody`.  

    <pre><code>
    sudo cat /etc/idmapd.conf
    # Example
    [General]
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
    mount 192.168.24.5:/sap<b>QAS</b>
    umount  /mnt/tmp
    echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping
    # Make the configuration permanent
    echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
    </code></pre>

   Para obtener más información sobre cómo cambiar el parámetro `nfs4_disable_idmapping`, consulte https://access.redhat.com/solutions/1749883.

### <a name="create-pacemaker-cluster"></a>Creación del clúster de Pacemaker

Siga los pasos de [Configuración de Pacemaker en Red Hat Enterprise Linux en Azure](high-availability-guide-rhel-pacemaker.md) para crear un clúster de Pacemaker básico para este servidor (A)SCS.

### <a name="prepare-for-sap-netweaver-installation"></a>Preparación de la instalación de SAP NetWeaver

Los elementos siguientes tienen el prefijo **[A]** : aplicable a todos los nodos, **[1]** : aplicable solo al nodo 1 o **[2]** : aplicable solo al nodo 2.

1. **[A]** Configure la resolución nombres de host

   Puede usar un servidor DNS o modificar /etc/hosts en todos los nodos. En este ejemplo se muestra cómo utilizar el archivo /etc/hosts.
   Reemplace la dirección IP y el nombre de host en los siguientes comandos

    ```
    sudo vi /etc/hosts
    ```

   Inserte las siguientes líneas en /etc/hosts. Cambie la dirección IP y el nombre de host para que coincida con su entorno

    ```
    # IP address of cluster node 1
    192.168.14.5    anftstsapcl1
    # IP address of cluster node 2
    192.168.14.6     anftstsapcl2
    # IP address of the load balancer frontend configuration for SAP Netweaver ASCS
    192.168.14.9    anftstsapvh
    # IP address of the load balancer frontend configuration for SAP Netweaver ERS
    192.168.14.10    anftstsapers
    ```

1. **[1]**  Cree directorios de SAP en el volumen de Azure NetApp Files.  
   Monte el volumen de Azure NetApp Files temporalmente en una de las máquinas virtuales y cree los directorios de SAP (rutas de acceso de archivos).  

    ```
     # mount temporarily the volume
     sudo mkdir -p /saptmp
     # If using NFSv3
     sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=3,tcp 192.168.24.5:/sapQAS /saptmp
     # If using NFSv4.1
     sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys,tcp 192.168.24.5:/sapQAS /saptmp
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

1. **[A]** Cree los directorios compartidos

   ```
   sudo mkdir -p /sapmnt/QAS
   sudo mkdir -p /usr/sap/trans
   sudo mkdir -p /usr/sap/QAS/SYS
   sudo mkdir -p /usr/sap/QAS/ASCS00
   sudo mkdir -p /usr/sap/QAS/ERS01
   
   sudo chattr +i /sapmnt/QAS
   sudo chattr +i /usr/sap/trans
   sudo chattr +i /usr/sap/QAS/SYS
   sudo chattr +i /usr/sap/QAS/ASCS00
   sudo chattr +i /usr/sap/QAS/ERS01
   ```

1. **[A]** Instalación del cliente de NFS y otros requisitos

   ```
   sudo yum -y install nfs-utils resource-agents resource-agents-sap
   ```

1. **[A]** Versión de comprobación de resource-agents-sap

   Asegúrese de que la versión del paquete resource-agents-sap instalado sea al menos 3.9.5-124.el7
   ```
   sudo yum info resource-agents-sap
   
   # Loaded plugins: langpacks, product-id, search-disabled-repos
   # Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
   # Installed Packages
   # Name        : resource-agents-sap
   # Arch        : x86_64
   # Version     : 3.9.5
   # Release     : 124.el7
   # Size        : 100 k
   # Repo        : installed
   # From repo   : rhel-sap-for-rhel-7-server-rpms
   # Summary     : SAP cluster resource agents and connector script
   # URL         : https://github.com/ClusterLabs/resource-agents
   # License     : GPLv2+
   # Description : The SAP resource agents and connector script interface with
   #          : Pacemaker to allow SAP instances to be managed in a cluster
   #          : environment.
   ```


1. **[A]** Adición de entradas de montaje

   Si usa NFSv3:
   ```
   sudo vi /etc/fstab
   
   # Add the following lines to fstab, save and exit
    192.168.24.5:/sapQAS/sapmntQAS /sapmnt/QAS nfs rw,hard,rsize=65536,wsize=65536,vers=3
    192.168.24.5:/sapQAS/usrsapQASsys /usr/sap/QAS/SYS nfs rw,hard,rsize=65536,wsize=65536,vers=3
    192.168.24.4:/transSAP /usr/sap/trans nfs rw,hard,rsize=65536,wsize=65536,vers=3
   ```

   Si usa NFSv4.1:
   ```
   sudo vi /etc/fstab
   
   # Add the following lines to fstab, save and exit
    192.168.24.5:/sapQAS/sapmntQAS /sapmnt/QAS nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
    192.168.24.5:/sapQAS/usrsapQASsys /usr/sap/QAS/SYS nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
    192.168.24.4:/transSAP /usr/sap/trans nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   ```

   > [!NOTE]
   > Asegúrese de hacer coincidir la versión del protocolo NFS de los volúmenes Azure NetApp Files al montar los volúmenes. Si los volúmenes de Azure NetApp Files se crean como volúmenes NFSv3, use la configuración de NFSv3 correspondiente. Si los volúmenes de Azure NetApp Files se crean como volúmenes NFSv4.1, siga las instrucciones para deshabilitar la asignación de Id. y asegúrese de usar la configuración de NFSv4.1 correspondiente. En este ejemplo, los volúmenes de Azure NetApp Files se crearon como volúmenes NFSv3.  

   Monte los nuevos recursos compartidos

   ```
   sudo mount -a  
   ```

1. **[A]** Configure el archivo de intercambio

   ```
   sudo vi /etc/waagent.conf
   
   # Set the property ResourceDisk.EnableSwap to y
   # Create and use swapfile on resource disk.
   ResourceDisk.EnableSwap=y
   
   # Set the size of the SWAP file with property ResourceDisk.SwapSizeMB
   # The free space of resource disk varies by virtual machine size. Make sure that you do not set a value that is too big. You can check the SWAP space with command swapon
   # Size of the swapfile.
   ResourceDisk.SwapSizeMB=2000
   ```

   Reinicie el agente para activar el cambio

   ```
   sudo service waagent restart
   ```

1. **[A]** Configuración de RHEL

   Configure RHEL tal como se describe en la nota de SAP [2002167].

### <a name="installing-sap-netweaver-ascsers"></a>Instalación de SAP NetWeaver ASCS/ERS

1. **[1]** Cree un recurso IP virtual y un sondeo de estado para la instancia de ASCS

   ```
   sudo pcs node standby anftstsapcl2
   # If using NFSv3
   sudo pcs resource create fs_QAS_ASCS Filesystem device='192.168.24.5:/sapQAS/usrsapQASascs' \
     directory='/usr/sap/QAS/ASCS00' fstype='nfs' force_unmount=safe \
     op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-QAS_ASCS
   
   # If using NFSv4.1
   sudo pcs resource create fs_QAS_ASCS Filesystem device='192.168.24.5:/sapQAS/usrsapQASascs' \
     directory='/usr/sap/QAS/ASCS00' fstype='nfs' force_unmount=safe options='sec=sys,vers=4.1' \
     op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-QAS_ASCS
   
   sudo pcs resource create vip_QAS_ASCS IPaddr2 \
     ip=192.168.14.9 cidr_netmask=24 \
     --group g-QAS_ASCS
   
   sudo pcs resource create nc_QAS_ASCS azure-lb port=62000 \
     --group g-QAS_ASCS
   ```

   Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.

   ```
   sudo pcs status
   
   # Node anftstsapcl2: standby
   # Online: [ anftstsapcl1 ]
   #
   # Full list of resources:
   #
   # rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
   #  Resource Group: g-QAS_ASCS
   #      fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
   #      nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
   #      vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
   ```

1. **[1]** Instale SAP NetWeaver ASCS  

   Instale SAP NetWeaver ASCS como raíz en el primer nodo mediante un nombre de host virtual que se asigna a la dirección IP de la configuración de front-end del equilibrador de carga para ASCS; por ejemplo, <b>anftstsapvh</b>, <b>192.168.14.9</b>. Especifique también el número de instancia que usó para el sondeo del equilibrador de carga; por ejemplo, <b>00</b>.

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst.

   ```
   # Allow access to SWPM. This rule is not permanent. If you reboot the machine, you have to run the command again.
   sudo firewall-cmd --zone=public  --add-port=4237/tcp
   
   sudo <swpm>/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=<virtual_hostname>
   ```

   Si se produce un error en la instalación para crear una subcarpeta en /usr/sap/**QAS**/ASCS**00**, pruebe a establecer el propietario y el grupo de la carpeta ASCS**00** e inténtelo de nuevo.

   ```
   sudo chown qasadm /usr/sap/QAS/ASCS00
   sudo chgrp sapsys /usr/sap/QAS/ASCS00
   ```

1. **[1]** Cree un recurso IP virtual y un sondeo de estado para la instancia de ERS

   ```
   sudo pcs node unstandby anftstsapcl2
   sudo pcs node standby anftstsapcl1
   
   # If using NFSv3
   sudo pcs resource create fs_QAS_AERS Filesystem device='192.168.24.5:/sapQAS/usrsapQASers' \
     directory='/usr/sap/QAS/ERS01' fstype='nfs' force_unmount=safe \
     op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
    --group g-QAS_AERS
   
   # If using NFSv4.1
   sudo pcs resource create fs_QAS_AERS Filesystem device='192.168.24.5:/sapQAS/usrsapQASers' \
     directory='/usr/sap/QAS/ERS01' fstype='nfs' force_unmount=safe options='sec=sys,vers=4.1' \
     op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
    --group g-QAS_AERS
   
   sudo pcs resource create vip_QAS_AERS IPaddr2 \
     ip=192.168.14.10 cidr_netmask=24 \
    --group g-QAS_AERS
   
   sudo pcs resource create nc_QAS_AERS azure-lb port=62101 \
    --group g-QAS_AERS
   ```
 
   Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.

   ```
   sudo pcs status
   
   # Node anftstsapcl1: standby
   # Online: [ anftstsapcl2 ]
   #
   # Full list of resources:
   #
   # rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl2
   #  Resource Group: g-QAS_ASCS
   #      fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
   #      nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2<
   #      vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
   #  Resource Group: g-QAS_AERS
   #      fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
   #      nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
   #      vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
   ```

1. **[2]** Instale SAP NetWeaver ERS  

   Instale SAP NetWeaver ERS como raíz en el segundo nodo mediante un nombre de host virtual que se asigna a la dirección IP de la configuración de front-end del equilibrador de carga para ERS (por ejemplo, <b>anftstsapers</b>, <b>192.168.14.10</b>) y el número de instancia que usó para el sondeo del equilibrador de carga (por ejemplo, <b>01</b>).

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst.

   ```
   # Allow access to SWPM. This rule is not permanent. If you reboot the machine, you have to run the command again.
   sudo firewall-cmd --zone=public  --add-port=4237/tcp

   sudo <swpm>/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=<virtual_hostname>
   ```

   Si se produce un error en la instalación para crear una subcarpeta en /usr/sap/**QAS**/ERS**01**, pruebe a establecer el propietario y el grupo de la carpeta ERS**01** e inténtelo de nuevo.

   ```
   sudo chown qaadm /usr/sap/QAS/ERS01
   sudo chgrp sapsys /usr/sap/QAS/ERS01
   ```

1. **[1]** Adapte los perfiles de instancias ASCS/SCS y ERS

   * Perfil ASCS/SCS

   ```
   sudo vi /sapmnt/QAS/profile/QAS_ASCS00_anftstsapvh
   
   # Change the restart command to a start command
   #Restart_Program_01 = local $(_EN) pf=$(_PF)
   Start_Program_01 = local $(_EN) pf=$(_PF)
   
   # Add the keep alive parameter, if using ENSA1
   enque/encni/set_so_keepalive = true
   ```

   En el caso de ENSA1 y ENSA2, asegúrese de que los parámetros del sistema operativo `keepalive` se establecen tal y como se describe en la nota de SAP [1410736](https://launchpad.support.sap.com/#/notes/1410736).  

   * Perfil ERS

   ```
   sudo vi /sapmnt/QAS/profile/QAS_ERS01_anftstsapers
   
   # Change the restart command to a start command
   #Restart_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   Start_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   
   # remove Autostart from ERS profile
   # Autostart = 1
   ```


1. **[A]** Configure la conexión persistente

   La comunicación entre el servidor de aplicaciones de SAP NetWeaver y ASCS/SCS se enruta a través de un equilibrador de carga de software. El equilibrador de carga desconecta las conexiones inactivas después de un tiempo de expiración que se puede configurar. Para evitar esto, debe establecer un parámetro en el perfil de SAP NetWeaver ASCS/SCS y cambiar la configuración del sistema Linux. Para más información, lea la [nota de SAP 1410736][1410736].

   ```
   # Change the Linux system configuration
   sudo sysctl net.ipv4.tcp_keepalive_time=120
   ```

1. **[A]** Actualización del archivo /usr/sap/sapservices

   Para evitar que el script de arranque sapinit inicie las instancias, todas las que sean administradas por Pacemaker se deben marcar como comentario en el archivo /usr/sap/sapservices. No marque como comentario la instancia de SAP HANA si se usará con HANA SR.

   ```
   sudo vi /usr/sap/sapservices
   
   # On the node where you installed the ASCS, comment out the following line
   # LD_LIBRARY_PATH=/usr/sap/QAS/ASCS00/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/QAS/ASCS00/exe/sapstartsrv pf=/usr/sap/QAS/SYS/profile/QAS_ASCS00_anftstsapvh -D -u qasadm
   
   # On the node where you installed the ERS, comment out the following line
   # LD_LIBRARY_PATH=/usr/sap/QAS/ERS01/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/QAS/ERS01/exe/sapstartsrv pf=/usr/sap/QAS/ERS01/profile/QAS_ERS01_anftstsapers -D -u qasadm
   ```

1. **[1]** Cree los recursos de clúster de SAP

   Si usa la arquitectura de servidor 1 de puesta en cola (ENSA1), defina los recursos tal como se indica:

   ```
   sudo pcs property set maintenance-mode=true
   
    sudo pcs resource create rsc_sap_QAS_ASCS00 SAPInstance \
    InstanceName=QAS_ASCS00_anftstsapvh START_PROFILE="/sapmnt/QAS/profile/QAS_ASCS00_anftstsapvh" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
    --group g-QAS_ASCS
   
    sudo pcs resource create rsc_sap_QAS_ERS01 SAPInstance \
    InstanceName=QAS_ERS01_anftstsapers START_PROFILE="/sapmnt/QAS/profile/QAS_ERS01_anftstsapers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
    --group g-QAS_AERS
      
    sudo pcs constraint colocation add g-QAS_AERS with g-QAS_ASCS -5000
    sudo pcs constraint location rsc_sap_QAS_ASCS00 rule score=2000 runs_ers_QAS eq 1
    sudo pcs constraint order g-QAS_ASCS then g-QAS_AERS kind=Optional symmetrical=false
    
    sudo pcs node unstandby anftstsapcl1
    sudo pcs property set maintenance-mode=false
    ```

   SAP introdujo una opción de compatibilidad con el servidor 2 de puesta en cola, incluida la replicación, a partir de la versión de SAP NW 7.52. A partir de la plataforma ABAP (versión 1809), el servidor 2 de puesta en cola está instalado de forma predeterminada. Consulte la nota de SAP [2630416](https://launchpad.support.sap.com/#/notes/2630416) para consultar la compatibilidad con el servidor 2 de puesta en cola.
   Si usa la arquitectura del servidor 2 de puesta en cola ([ENSA2](https://help.sap.com/viewer/cff8531bc1d9416d91bb6781e628d4e0/1709%20001/en-US/6d655c383abf4c129b0e5c8683e7ecd8.html)), instale el agente de recursos resource-agents-sap-4.1.1-12.el7.x86_64 o una versión más reciente y defina los recursos tal como se indica:

    ```
    sudo pcs property set maintenance-mode=true
    
    sudo pcs resource create rsc_sap_QAS_ASCS00 SAPInstance \
    InstanceName=QAS_ASCS00_anftstsapvh START_PROFILE="/sapmnt/QAS/profile/QAS_ASCS00_anftstsapvh" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
    --group g-QAS_ASCS
   
    sudo pcs resource create rsc_sap_QAS_ERS01 SAPInstance \
    InstanceName=QAS_ERS01_anftstsapers START_PROFILE="/sapmnt/QAS/profile/QAS_ERS01_anftstsapers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
    --group g-QAS_AERS
      
    sudo pcs constraint colocation add g-QAS_AERS with g-QAS_ASCS -5000
    sudo pcs constraint order g-QAS_ASCS then g-QAS_AERS kind=Optional symmetrical=false
    sudo pcs constraint order start g-QAS_ASCS then stop g-QAS_AERS symmetrical=false
   
    sudo pcs node unstandby anftstsapcl1
    sudo pcs property set maintenance-mode=false
    ```

   Si está actualizando desde una versión anterior y va a cambiar al servidor 2 de puesta en cola, consulte la nota de SAP [2641322](https://launchpad.support.sap.com/#/notes/2641322). 

   > [!NOTE]
   > Los tiempos de espera de la configuración anterior son solo ejemplos y puede ser necesario adaptarlos a la configuración específica de SAP. 

   Asegúrese de que el estado del clúster sea el correcto y que se iniciaron todos los recursos. No es importante en qué nodo se ejecutan los recursos.

    ```
    sudo pcs status
    
    # Online: [ anftstsapcl1 anftstsapcl2 ]
    #
    # Full list of resources:
    #
    # rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl2
    #  Resource Group: g-QAS_ASCS
    #      fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
    #      nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
    #      vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
    #      rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    #  Resource Group: g-QAS_AERS
    #      fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
    #      nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
    #      vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
    #      rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

1. **[A]**  Agregar reglas de firewall para ASCS y ERS en ambos nodos Agregue las reglas de firewall para ASCS y ERS en ambos nodos.
   ```
   # Probe Port of ASCS
   sudo firewall-cmd --zone=public --add-port=62000/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=62000/tcp
   sudo firewall-cmd --zone=public --add-port=3200/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=3200/tcp
   sudo firewall-cmd --zone=public --add-port=3600/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=3600/tcp
   sudo firewall-cmd --zone=public --add-port=3900/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=3900/tcp
   sudo firewall-cmd --zone=public --add-port=8100/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=8100/tcp
   sudo firewall-cmd --zone=public --add-port=50013/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50013/tcp
   sudo firewall-cmd --zone=public --add-port=50014/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50014/tcp
   sudo firewall-cmd --zone=public --add-port=50016/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50016/tcp
   # Probe Port of ERS
   sudo firewall-cmd --zone=public --add-port=62101/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=62101/tcp
   sudo firewall-cmd --zone=public --add-port=3301/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=3301/tcp
   sudo firewall-cmd --zone=public --add-port=50113/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50113/tcp
   sudo firewall-cmd --zone=public --add-port=50114/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50114/tcp
   sudo firewall-cmd --zone=public --add-port=50116/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50116/tcp
   ```

## <a name="sap-netweaver-application-server-preparation"></a><a name="2d6008b0-685d-426c-b59e-6cd281fd45d7"></a>Preparación del servidor de aplicaciones de SAP NetWeaver

   Algunas bases de datos requieren que la instalación de la instancia de base de datos se ejecute en un servidor de aplicaciones. Prepare las máquinas virtuales del servidor de aplicaciones para poder usarlas en estos casos.  

   En los pasos siguientes se supone que instala el servidor de aplicaciones en un servidor distinto de los servidores ASCS/SCS y HANA. De lo contrario, no se necesitan algunos de los pasos que aparecen a continuación (como configurar la resolución de nombres de host).  

   Los elementos siguientes tienen el prefijo **[A]** (aplicable a PAS y AAS), **[P]** (solo aplicable a PAS) o **[S]** (solo aplicable a AAS).  

1. **[A]** Configurar la resolución del nombre del host Puede usar un servidor DNS o modificar /etc/hosts en todos los nodos. En este ejemplo se muestra cómo utilizar el archivo /etc/hosts.
   Reemplace la dirección IP y el nombre de host en los siguientes comandos:  

   ```
   sudo vi /etc/hosts
   ```

   Inserte las siguientes líneas en /etc/hosts. Cambie la dirección IP y el nombre de host para que coincidan con su entorno.

   ```
   # IP address of the load balancer frontend configuration for SAP NetWeaver ASCS
   192.168.14.9 anftstsapvh
   # IP address of the load balancer frontend configuration for SAP NetWeaver ASCS ERS
   192.168.14.10 anftstsapers
   192.168.14.7 anftstsapa01
   192.168.14.8 anftstsapa02
   ```

1. **[A]**  Crear el directorio sapmnt Cree el directorio sapmnt.
   ```
   sudo mkdir -p /sapmnt/QAS
   sudo mkdir -p /usr/sap/trans

   sudo chattr +i /sapmnt/QAS
   sudo chattr +i /usr/sap/trans
   ```

1. **[A]** Instalación del cliente de NFS y otros requisitos  

   ```
   sudo yum -y install nfs-utils uuidd
   ```

1. **[A]** Adición de entradas de montaje  
   Si usa NFSv3:
   ```
   sudo vi /etc/fstab
   
   # Add the following lines to fstab, save and exit
   192.168.24.5:/sapQAS/sapmntQAS /sapmnt/QAS nfs rw,hard,rsize=65536,wsize=65536,vers=3
   192.168.24.4:/transSAP /usr/sap/trans nfs rw,hard,rsize=65536,wsize=65536,vers=3
   ```

   Si usa NFSv4.1:
   ```
   sudo vi /etc/fstab
   
   # Add the following lines to fstab, save and exit
   192.168.24.5:/sapQAS/sapmntQAS /sapmnt/QAS nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   192.168.24.4:/transSAP /usr/sap/trans nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   ```

   Monte los nuevos recursos compartidos

   ```
   sudo mount -a
   ```

1. **[P]** Crear y montar el directorio PAS  
   Si usa NFSv3:
   ```
   sudo mkdir -p /usr/sap/QAS/D02
   sudo chattr +i /usr/sap/QAS/D02
   
   sudo vi /etc/fstab
   # Add the following line to fstab
   92.168.24.5:/sapQAS/usrsapQASpas /usr/sap/QAS/D02 nfs rw,hard,rsize=65536,wsize=65536,vers=3
   
   # Mount
   sudo mount -a
   ```

   Si usa NFSv4.1:
   ```
   sudo mkdir -p /usr/sap/QAS/D02
   sudo chattr +i /usr/sap/QAS/D02
   
   sudo vi /etc/fstab
   # Add the following line to fstab
   92.168.24.5:/sapQAS/usrsapQASpas /usr/sap/QAS/D02 nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   
   # Mount
   sudo mount -a
   ```

1. **[P]** Crear y montar el directorio AAS  
   Si usa NFSv3:
   ```
   sudo mkdir -p /usr/sap/QAS/D03
   sudo chattr +i /usr/sap/QAS/D03
   
   sudo vi /etc/fstab
   # Add the following line to fstab
   92.168.24.5:/sapQAS/usrsapQASaas /usr/sap/QAS/D03 nfs rw,hard,rsize=65536,wsize=65536,vers=3
   
   # Mount
   sudo mount -a
   ```

   Si usa NFSv4.1:
   ```
   sudo mkdir -p /usr/sap/QAS/D03
   sudo chattr +i /usr/sap/QAS/D03
   
   sudo vi /etc/fstab
   # Add the following line to fstab
   92.168.24.5:/sapQAS/usrsapQASaas /usr/sap/QAS/D03 nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   
   # Mount
   sudo mount -a
   ```

1. **[A]** Configure el archivo de intercambio
 
   ```
   sudo vi /etc/waagent.conf
   
   # Set the property ResourceDisk.EnableSwap to y
   # Create and use swapfile on resource disk.
   ResourceDisk.EnableSwap=y
   
   # Set the size of the SWAP file with property ResourceDisk.SwapSizeMB
   # The free space of resource disk varies by virtual machine size. Make sure that you do not set a value that is too big. You can check the SWAP space with command swapon
   # Size of the swapfile.
   ResourceDisk.SwapSizeMB=2000
   ```

   Reinicie el agente para activar el cambio

   ```
   sudo service waagent restart
   ```

## <a name="install-database"></a>Instalar la base de datos

En este ejemplo, SAP NetWeaver se instala en SAP HANA. En esta instalación puede usar todas las bases de datos admitidas. Para más información sobre cómo instalar SAP HANA en Azure, consulte [Alta disponibilidad de SAP HANA en Red Hat Enterprise Linux][sap-hana-ha]. For a list of supported databases, see [SAP Note 1928533][1928533].

1. Ejecute la instalación de la instancia de base de datos de SAP.

   Instale la instancia de base de datos de SAP NetWeaver como raíz con un nombre de host virtual que se asigna a la dirección IP de la configuración de front-end del equilibrador de carga para la base de datos.

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst.

   ```
   sudo <swpm>/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin
   ```

## <a name="sap-netweaver-application-server-installation"></a>Instalación del servidor de aplicaciones de SAP NetWeaver

Siga estos pasos para instalar un servidor de aplicaciones de SAP.

1. Preparación del servidor de aplicaciones

   Siga los pasos descritos en el capítulo [Preparación del servidor de aplicaciones de SAP NetWeaver](high-availability-guide-rhel.md#2d6008b0-685d-426c-b59e-6cd281fd45d7) anterior para preparar el servidor de aplicaciones.

1. Instale el servidor de aplicaciones de SAP NetWeaver

   Instale un servidor de aplicaciones de SAP NetWeaver principal o adicional.

   Puede usar el parámetro de sapinst SAPINST_REMOTE_ACCESS_USER para permitir que un usuario no raíz se conecta a sapinst.

   ```
   sudo <swpm>/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin
   ```

1. Actualice el almacenamiento seguro de SAP HANA

   Actualice el almacenamiento seguro de SAP HANA que apunte al nombre virtual de la configuración de la replicación del sistema SAP HANA.

   Ejecute el siguiente comando para mostrar las entradas como \<sapsid>adm

   ```
   hdbuserstore List
   ```

   Se deberían mostrar todas las entradas y deberían ser parecidas a estas:
   ```
   DATA FILE       : /home/qasadm/.hdb/anftstsapa01/SSFS_HDB.DAT
   KEY FILE        : /home/qasadm/.hdb/anftstsapa01/SSFS_HDB.KEY
   
   KEY DEFAULT
     ENV : 192.168.14.4:30313
     USER: SAPABAP1
     DATABASE: QAS
   ```

   El resultado muestra que la dirección IP de la entrada predeterminada apunta a la máquina virtual y no a la dirección IP del equilibrador de carga. Esta entrada debe modificarse para que apunte al nombre de host virtual del equilibrador de carga. Asegúrese de usar el mismo puerto (**30313** en la salida anterior) y el mismo nombre de base de datos (**QAS** en la salida anterior).

   ```
   su - qasadm
   hdbuserstore SET DEFAULT qasdb:30313@QAS SAPABAP1 <password of ABAP schema>
   ```

## <a name="test-the-cluster-setup"></a>Prueba de la configuración del clúster

1. Migración manual de la instancia de ASCS

   Estado del recurso antes de iniciar la prueba:

   ```
    rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

   Ejecute los siguientes comandos como raíz para migrar la instancia de ASCS.

   ```
   [root@anftstsapcl1 ~]# pcs resource move rsc_sap_QAS_ASCS00
   
   [root@anftstsapcl1 ~]# pcs resource clear rsc_sap_QAS_ASCS00
   
   # Remove failed actions for the ERS that occurred as part of the migration
   [root@anftstsapcl1 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   Estado del recurso después de la prueba:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

1. Simulación de bloqueo de nodo

   Estado del recurso antes de iniciar la prueba:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

   Ejecute el siguiente comando como raíz en el nodo donde se ejecuta la instancia de ASCS

   ```
   [root@anftstsapcl2 ~]# echo b > /proc/sysrq-trigger
   ```

   El estado después de que el nodo se inicia de nuevo debe parecerse a este.

   ```
   Online: [ anftstsapcl1 anftstsapcl2 ]
   
   Full list of resources:
   
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   
   Failed Actions:
   * rsc_sap_QAS_ERS01_monitor_11000 on anftstsapcl1 'not running' (7): call=45, status=complete, exitreason='',
   ```

   Use el siguiente comando para limpiar los recursos con error.

   ```
   [root@anftstsapcl1 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   Estado del recurso después de la prueba:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

1. Terminación del proceso del servidor de mensajes

   Estado del recurso antes de iniciar la prueba:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```
   
   Ejecute los siguientes comandos como raíz para identificar el proceso del servidor de mensajes y terminarlo.

   ```
   [root@anftstsapcl1 ~]# pgrep ms.sapQAS | xargs kill -9
   ```

   Si solo termina el servidor de mensajes una vez, se reiniciará mediante `sapstart`. Si lo termina con bastante frecuencia, Pacemaker acabará moviendo la instancia de ASCS al otro nodo. Ejecute los siguientes comandos como raíz para limpiar el estado del recurso de la instancia de ASCS y ERS después de la prueba.

   ```
   [root@anftstsapcl1 ~]# pcs resource cleanup rsc_sap_QAS_ASCS00
   [root@anftstsapcl1 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   Estado del recurso después de la prueba:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

1. Terminación del proceso del servidor de puesta en cola

   Estado del recurso antes de iniciar la prueba:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

   Ejecute los siguientes comandos como raíz en el nodo donde se ejecuta la instancia de ASCS para terminar el servidor de puesta en cola.

   ```
   [root@anftstsapcl2 ~]# pgrep en.sapQAS | xargs kill -9
   ```

   La instancia de ASCS conmutará inmediatamente por error al otro nodo. La instancia de ERS también conmutará por error después de iniciarse la instancia de ASCS. Ejecute los siguientes comandos como raíz para limpiar el estado del recurso de la instancia de ASCS y ERS después de la prueba.

   ```
   [root@anftstsapcl2 ~]# pcs resource cleanup rsc_sap_QAS_ASCS00
   [root@anftstsapcl2 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   Estado del recurso después de la prueba:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

1. Terminación del proceso del servidor de replicación

   Estado del recurso antes de iniciar la prueba:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

   Ejecute el siguiente comando como raíz en el nodo donde se ejecuta la instancia de ERS para terminar el proceso del servidor de replicación de puesta en cola.

   ```
   [root@anftstsapcl2 ~]# pgrep er.sapQAS | xargs kill -9
   ```

   Si solo ejecuta el comando una vez, `sapstart` reiniciará el proceso. Si lo ejecuta con bastante frecuencia, `sapstart` no reiniciará el proceso y el recurso estará en estado detenido. Ejecute los siguientes comandos como raíz para limpiar el estado del recurso de la instancia de ERS después de la prueba.

   ```
   [root@anftstsapcl2 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   Estado del recurso después de la prueba:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

1. Terminación del proceso sapstartsrv de puesta en cola

   Estado del recurso antes de iniciar la prueba:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

   Ejecute los siguientes comandos como raíz en el nodo donde se ejecuta ASCS.

   ```
   [root@anftstsapcl1 ~]# pgrep -fl ASCS00.*sapstartsrv
   # 59545 sapstartsrv
   
   [root@anftstsapcl1 ~]# kill -9 59545
   ```

   El proceso sapstartsrv siempre se debe reiniciar con el agente de recursos de Pacemaker como parte de la supervisión. Estado del recurso después de la prueba:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

## <a name="next-steps"></a>Pasos siguientes

* [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en Red Hat Enterprise Linux para SAP Applications: guía de varios SID](./high-availability-guide-rhel-multi-sid.md)
* [Planeamiento e implementación de Azure Virtual Machines para SAP][planning-guide]
* [Implementación de Azure Virtual Machines para SAP][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP][dbms-guide]
* Para obtener información sobre cómo establecer la alta disponibilidad y planear la recuperación ante desastres de SAP HANA en Azure (instancias grandes), vea [Alta disponibilidad y recuperación ante desastres de SAP HANA en Azure (instancias grandes)](hana-overview-high-availability-disaster-recovery.md).
* Para más información sobre cómo establecer la alta disponibilidad y planear la recuperación ante desastres de SAP HANA en Azure Virtual Machines, consulte [Alta disponibilidad de SAP HANA en las máquinas virtuales de Azure][sap-hana-ha].
