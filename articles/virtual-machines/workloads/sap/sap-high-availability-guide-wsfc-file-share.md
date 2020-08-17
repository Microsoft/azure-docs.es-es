---
title: Agrupación de ASCS/SCS de SAP en WSFC con un recurso compartido de archivos en Azure | Microsoft Docs
description: Aprenda a agrupar una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un recurso compartido de archivos en Azure.
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
ms.date: 07/24/2019
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: bf9e00e8acba241f1445977dcc53724b9981039f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87068697"
---
# <a name="cluster-an-sap-ascsscs-instance-on-a-windows-failover-cluster-by-using-a-file-share-in-azure"></a>Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un recurso compartido de archivos en Azure

> ![Windows][Logo_Windows] Windows
>

Los clústeres de conmutación por error de Windows Server son la base de una instalación de ASCS/SCS de SAP de alta disponibilidad y DBMS en Windows.

Un clúster de conmutación por error es un grupo de 1+n servidores independientes (nodos) que colaboran para aumentar la disponibilidad de aplicaciones y servicios. Si se produce un error de nodo, los clústeres de conmutación por error de Windows Server calculan el número de errores que se pueden producir y mantiene un clúster en buen estado para proporcionar aplicaciones y servicios. Para conseguir clústeres de conmutación por error, puede elegir entre distintos modos de cuórum.

## <a name="prerequisites"></a>Requisitos previos
Antes de comenzar las tareas que se describen en este artículo, consulte este otro artículo:

* [Escenarios y arquitectura de alta disponibilidad de Azure Virtual Machines para SAP NetWeaver][sap-high-availability-architecture-scenarios]

> [!IMPORTANT]
> La agrupación en clústeres de instancias de ASCS/SCS de SAP con un recurso compartido de archivos es compatible con SAP NetWeaver 7.40 (y versiones posteriores), con el kernel de SAP 7.49 (y versiones posteriores).
>


## <a name="windows-server-failover-clustering-in-azure"></a>Clústeres de conmutación por error de Windows Server en Azure

En comparación con las implementaciones de nube privada o físicas, Azure Virtual Machines requiere pasos adicionales para configurar clústeres de conmutación por error de Windows Server. Al crear un clúster, debe establecer varias direcciones IP y nombres de host virtual para la instancia de ASCS/SCS de SAP.

### <a name="name-resolution-in-azure-and-the-cluster-virtual-host-name"></a>Resolución de nombres en Azure y el nombre de host virtual del clúster

La plataforma en la nube de Azure no ofrece la opción de configurar direcciones IP virtuales, como direcciones IP flotantes. Necesita una solución alternativa para configurar una dirección IP virtual que se comunique con el recurso de clúster en la nube. 

El servicio Azure Load Balancer proporciona un *equilibrador de carga interno* para Azure. Con el equilibrador de carga interno, los clientes se comunican con el clúster a través de la dirección IP virtual del clúster. 

Implemente el equilibrador de carga interno en el grupo de recursos que contiene los nodos del clúster. A continuación, configure todas las reglas de reenvío de puertos necesarias utilizando los puertos de sondeo del equilibrador de carga interno. Los clientes se pueden conectar por medio del nombre de host virtual. El servidor DNS resuelve la dirección IP del clúster. El equilibrador de carga interno controla el reenvío de puertos al nodo activo del clúster.

![Figura 1: Configuración de clústeres de conmutación por error de Windows Server en Azure sin un disco compartido][sap-ha-guide-figure-1001]

_**Ilustración 1:** Configuración de clústeres de conmutación por error de Windows Server en Azure sin un disco compartido_

## <a name="sap-ascsscs-ha-with-file-share"></a>Alta disponibilidad de ASCS/SCS de SAP con recurso compartido de archivos

SAP ha desarrollado un nuevo enfoque y alternativas para agrupar en clústeres los discos compartidos, a fin de agrupar una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows. En lugar de utilizar discos compartidos de clúster, puede usar un recurso compartido de archivos SMB para implementar los archivos del host global de SAP.

> [!NOTE]
> Un recurso compartido de archivos SMB es una alternativa al uso de discos compartidos de clúster para agrupar en clústeres instancias de ASCS/SCS de SAP.  
>

Esta arquitectura es específica de las maneras siguientes:

* Los servicios centrales de SAP (con su propia estructura de archivos y procesos de mensajería y puesta en cola) están separados de los archivos del host global de SAP.
* Los servicios centrales de SAP se ejecutan en una instancia de ASCS/SCS de SAP.
* La instancia de ASCS/SCS de SAP está en clúster y es accesible mediante el nombre de host virtual \<ASCS/SCS virtual host name\>.
* Los archivos globales de SAP se colocan en el recurso compartido de archivos SMB y son accesibles mediante el nombre de host \<SAP global host\>: \\\\&lt;host global de SAP&gt;\sapmnt\\&lt;SID&gt;\SYS\..
* La instancia de ASCS/SCS de SAP está instalada en un disco local en ambos nodos del clúster.
* El nombre de red \<ASCS/SCS virtual host name\> es diferente del &lt;host global de SAP&gt;.

![Ilustración 2: Arquitectura de alta disponibilidad de ASCS/SCS de SAP con recurso compartido de archivos SMB][sap-ha-guide-figure-8004]

_**Ilustración 2:** Arquitectura de alta disponibilidad de ASCS/SCS de SAP nueva con un recurso compartido de archivos SMB_

Requisitos previos para un recurso compartido de archivos SMB:

* Protocolo SMB 3.0 (o posterior).
* Capacidad de establecer listas de control de acceso (ACL) de Active Directory para grupos de usuarios de Active Directory y el objeto de equipo `computer$`.
* El recurso compartido de archivos debe estar habilitado para alta disponibilidad:
    * Los discos usados para almacenar archivos no deben ser un único punto de error.
    * El tiempo de inactividad del servidor o de la máquina virtual no provoca tiempo de inactividad en el recurso compartido de archivos.

El rol de clúster \<SID\> de SAP no contiene discos compartidos de clúster ni un recurso compartido de archivos de clúster genérico.


![Ilustración 3: Recursos del rol de clúster \<SID\> de SAP cuando se usa un recurso compartido de archivos][sap-ha-guide-figure-8005]

_**Ilustración 3:** Recursos del rol de clúster &lt;SID&gt; de SAP cuando se usa un recurso compartido de archivos_


## <a name="scale-out-file-shares-with-storage-spaces-direct-in-azure-as-an-sapmnt-file-share"></a>Recursos compartidos de archivos de escalabilidad horizontal con espacios de almacenamiento directo en Azure como un recurso compartido de archivos SAPMNT

Puede usar un recurso compartido de archivos de escalabilidad horizontal para hospedar y proteger los archivos del host global de SAP. Un recurso compartido de archivos de escalabilidad horizontal también ofrece un servicio de recurso compartido de archivos SAPMNT de alta disponibilidad.

![Ilustración 4: Recurso compartido de archivos de escalabilidad horizontal utilizado para proteger los archivos del host global de SAP][sap-ha-guide-figure-8006]

_**Ilustración 4:** Recurso compartido de archivos de escalabilidad horizontal utilizado para proteger los archivos del host global de SAP_

> [!IMPORTANT]
> Los recursos compartidos de archivos de escalabilidad horizontal son totalmente compatibles con la nube de Microsoft Azure y en entornos locales.
>

Un recurso compartido de archivos de escalabilidad horizontal ofrece un recurso compartido de archivos SAPMNT de alta disponibilidad y escalado horizontal.

Los espacios de almacenamiento directo se usan como un disco compartido para un recurso compartido de archivos de escalabilidad horizontal. Puede usar espacios de almacenamiento directo para crear almacenamiento de alta disponibilidad y escalabilidad utilizando servidores con almacenamiento local. El almacenamiento compartido que se utiliza para un recurso compartido de archivos de escalabilidad horizontal, al igual que para los archivos del host global de SAP, no es un punto único de error.

Al elegir Espacios de almacenamiento directo, tenga en cuenta estos casos de uso:

- Las máquinas virtuales que se usan para compilar el clúster de Espacios de almacenamiento directo deben implementarse en un conjunto de disponibilidad de Azure.
- Para la recuperación ante desastres de un clúster de Espacios de almacenamiento directo, puede usar [Azure Site Recovery Services](../../../site-recovery/azure-to-azure-support-matrix.md#replicated-machines---storage).
- No se admite expandir el clúster de Espacios de almacenamiento directo entre distintas instancias de Azure Availability Zones.

### <a name="sap-prerequisites-for-scale-out-file-shares-in-azure"></a>Requisitos previos de SAP para recursos compartidos de archivos de escalabilidad horizontal en Azure

Para usar un recurso compartido de archivos de escalabilidad horizontal, el sistema debe cumplir los siguientes requisitos:

* Al menos dos nodos del clúster para un recurso compartido de archivos de escalabilidad horizontal.
* Cada nodo debe tener al menos dos discos locales.
* Por razones de rendimiento, debe usar la *resistencia de creación de reflejo*:
    * Creación de reflejo bidireccional para un recurso compartido de archivos de escalabilidad horizontal con dos nodos del clúster.
    * Creación de reflejo triple para un recurso compartido de archivos de escalabilidad horizontal con tres (o más) nodos del clúster.
* Se recomiendan tres (o más) nodos del clúster para un recurso compartido de archivos de escalabilidad horizontal con creación de reflejo triple.
    Esta configuración ofrece más escalabilidad y resistencia de almacenamiento que la configuración de recurso compartido de archivos de escalabilidad horizontal con dos nodos del clúster y creación de reflejo bidireccional.
* Debe usar discos Premium de Azure.
* Se recomienda utilizar Azure Managed Disks.
* Se recomienda dar formato a los volúmenes con el sistema de archivos resistente (ReFS).
    * Para más información, consulte la [Nota de SAP 1869038: compatibilidad de SAP para el sistema de archivos ReFs][1869038] y el capítulo [Elección del sistema de archivos][planning-volumes-s2d-choosing-filesystem] de artículo Planificación de volúmenes en espacios de almacenamiento directo.
    * Asegúrese de que ha instalado la [actualización acumulativa de Microsoft KB4025334][kb4025334].
* Puede usar los tamaños de máquina virtual de Azure de la serie DS o la serie DSv2.
* Para un buen rendimiento de red entre las máquinas virtuales, lo cual es necesario para la sincronización de disco de los espacios de almacenamiento directo, utilice un tipo de máquina virtual con un ancho de banda de red "alto" como mínimo.
    Para más información, consulte las especificaciones de la [serie DSv2][dv2-series] y la [serie DS][ds-series].
* Se recomienda reservar cierta capacidad sin asignar en el grupo de almacenamiento. Dejar cierta capacidad sin asignar en el grupo de almacenamiento da espacio a los volúmenes para una reparación "in situ" si se produce un error en una unidad. Esto mejora el rendimiento y seguridad de los datos.  Para obtener más información, vea [Elección del tamaño del volumen][choosing-the-size-of-volumes-s2d].
* No es necesario configurar el equilibrador de carga interno de Azure para el nombre de red del recurso compartido de archivos de escalabilidad horizontal, como para el \<SAP global host\>. Esto se hace para el \<ASCS/SCS virtual host name\> de la instancia de ASCS/SCS de SAP o para el DBMS. Un recurso compartido de archivos de escalabilidad horizontal escala horizontalmente la carga entre todos los nodos del clúster. El \<SAP global host\> usa la dirección IP local para todos los nodos del clúster.


> [!IMPORTANT]
> No se puede cambiar el nombre del recurso compartido de archivos SAPMNT, que apunta al \<SAP global host\>. SAP admite únicamente el nombre de recurso compartido "sapmnt."
>
> Para obtener más información, vea [Nota de SAP 2492395: ¿Se puede cambiar el nombre de recurso compartido sapmnt?][2492395]

### <a name="configure-sap-ascsscs-instances-and-a-scale-out-file-share-in-two-clusters"></a>Configuración de instancias de ASCS/SCS de SAP y un recurso compartido de archivos de escalabilidad horizontal en dos clústeres

Puede implementar instancias de ASCS/SCS de SAP en un clúster, con su propio rol de clúster \<SID\> de SAP. En este caso, configure el recurso compartido de archivos de escalabilidad horizontal en otro clúster con otro rol de clúster.

> [!IMPORTANT]
>En este escenario, la instancia de ASCS/SCS de SAP se configura para tener acceso al host global de SAP mediante la ruta de acceso UNC \\\\&lt;host global de SAP&gt;\sapmnt\\&lt;SID&gt;\SYS\.
>

![Ilustración 5: Instancia de ASCS/SCS de SAP y un recurso compartido de archivos de escalabilidad horizontal implementados en dos clústeres][sap-ha-guide-figure-8007]

_**Ilustración 5:** Instancia de ASCS/SCS de SAP y un recurso compartido de archivos de escalabilidad horizontal implementados en dos clústeres_

> [!IMPORTANT]
> En la nube de Azure, los clústeres que se usan para SAP y los recursos compartidos de archivos de escalabilidad horizontal deben implementarse en su propio conjunto de disponibilidad de Azure o entre instancias de Azure Availability Zones. Esto garantiza la selección de ubicación distribuida de las máquinas virtuales del clúster a través de la infraestructura subyacente de Azure. Las implementaciones de zonas de disponibilidad son compatibles con esta tecnología.
>

## <a name="generic-file-share-with-sios-datakeeper-as-cluster-shared-disks"></a>Recurso compartido de archivos genérico con SIOS DataKeeper como discos compartidos de clúster


Otra opción para conseguir un recurso compartido de archivos de alta disponibilidad es un recurso compartido de archivos genérico.

En este caso, puede usar una solución SIOS de terceros como un disco compartido de clúster.

## <a name="next-steps"></a>Pasos siguientes

* [Preparación de la infraestructura de Azure para alta disponibilidad de SAP con un clúster de conmutación por error de Windows y un recurso compartido de archivos para una instancia de ASCS/SCS de SAP][sap-high-availability-infrastructure-wsfc-file-share]
* [Instalación de alta disponibilidad para SAP NetWeaver en un clúster de conmutación por error de Windows y un recurso compartido de archivos para una instancia de ASCS/SCS de SAP][sap-high-availability-installation-wsfc-shared-disk]
* [Implementar un servidor de archivos de escalado horizontal de Espacios de almacenamiento directo de dos nodos para el almacenamiento UPD en Azure][deploy-sofs-s2d-in-azure]
* [Espacios de almacenamiento directo en Windows Server 2016][s2d-in-win-2016]
* [En profundidad: volúmenes en Espacios de almacenamiento directo][deep-dive-volumes-in-s2d]

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[2492395]:https://launchpad.support.sap.com/#/notes/2492395

[kb4025334]:https://support.microsoft.com/help/4025334/windows-10-update-kb4025334

[dv2-series]:../../dv2-dsv2-series.md
[ds-series]:https://docs.microsoft.com/azure/virtual-machines/windows/sizes-general

[sap-installation-guides]:http://service.sap.com/instguides

[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md

[dbms-guide]:../../virtual-machines-windows-sap-dbms-guide.md

[deployment-guide]:deployment-guide.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md

[ha-guide]:sap-high-availability-guide.md
[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-file-share]:sap-high-availability-infrastructure-wsfc-file-share.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md

[planning-volumes-s2d-choosing-filesystem]:https://docs.microsoft.com/windows-server/storage/storage-spaces/plan-volumes#choosing-the-filesystem
[choosing-the-size-of-volumes-s2d]:https://docs.microsoft.com/windows-server/storage/storage-spaces/plan-volumes#choosing-the-size-of-volumes
[deploy-sofs-s2d-in-azure]:https://docs.microsoft.com/windows-server/remote/remote-desktop-services/rds-storage-spaces-direct-deployment
[s2d-in-win-2016]:https://docs.microsoft.com/windows-server/storage/storage-spaces/storage-spaces-direct-overview
[deep-dive-volumes-in-s2d]:https://blogs.technet.microsoft.com/filecab/2016/08/29/deep-dive-volumes-in-spaces-direct/

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

[sap-ha-guide]:sap-high-availability-guide.md
[sap-ha-guide-2]:#42b8f600-7ba3-4606-b8a5-53c4f026da08
[sap-ha-guide-4]:#8ecf3ba0-67c0-4495-9c14-feec1a2255b7
[sap-ha-guide-8]:#78092dbe-165b-454c-92f5-4972bdbef9bf
[sap-ha-guide-8.1]:#c87a8d3f-b1dc-4d2f-b23c-da4b72977489
[sap-ha-guide-8.9]:#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-ha-guide-8.11]:#661035b2-4d0f-4d31-86f8-dc0a50d78158
[sap-ha-guide-8.12]:#0d67f090-7928-43e0-8772-5ccbf8f59aab
[sap-ha-guide-8.12.1]:#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79
[sap-ha-guide-8.12.3]:#5c8e5482-841e-45e1-a89d-a05c0907c868
[sap-ha-guide-8.12.3.1]:#1c2788c3-3648-4e82-9e0d-e058e475e2a3
[sap-ha-guide-8.12.3.2]:#dd41d5a2-8083-415b-9878-839652812102
[sap-ha-guide-8.12.3.3]:#d9c1fc8e-8710-4dff-bec2-1f535db7b006
[sap-ha-guide-9]:#a06f0b49-8a7a-42bf-8b0d-c12026c5746b
[sap-ha-guide-9.1]:#31c6bd4f-51df-4057-9fdf-3fcbc619c170
[sap-ha-guide-9.1.1]:#a97ad604-9094-44fe-a364-f89cb39bf097

[sap-ha-multi-sid-guide]:sap-high-availability-multi-sid.md (Configuración de alta disponibilidad de varios SID de SAP)

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png

[sap-ha-guide-figure-1000]:./media/virtual-machines-shared-sap-high-availability-guide/1000-wsfc-for-sap-ascs-on-azure.png
[sap-ha-guide-figure-1001]:./media/virtual-machines-shared-sap-high-availability-guide/1001-wsfc-on-azure-ilb.png
[sap-ha-guide-figure-1002]:./media/virtual-machines-shared-sap-high-availability-guide/1002-wsfc-sios-on-azure-ilb.png
[sap-ha-guide-figure-2000]:./media/virtual-machines-shared-sap-high-availability-guide/2000-wsfc-sap-as-ha-on-azure.png
[sap-ha-guide-figure-2001]:./media/virtual-machines-shared-sap-high-availability-guide/2001-wsfc-sap-ascs-ha-on-azure.png
[sap-ha-guide-figure-2003]:./media/virtual-machines-shared-sap-high-availability-guide/2003-wsfc-sap-dbms-ha-on-azure.png
[sap-ha-guide-figure-2004]:./media/virtual-machines-shared-sap-high-availability-guide/2004-wsfc-sap-ha-e2e-archit-template1-on-azure.png
[sap-ha-guide-figure-2005]:./media/virtual-machines-shared-sap-high-availability-guide/2005-wsfc-sap-ha-e2e-arch-template2-on-azure.png

[sap-ha-guide-figure-3000]:./media/virtual-machines-shared-sap-high-availability-guide/3000-template-parameters-sap-ha-arm-on-azure.png
[sap-ha-guide-figure-3001]:./media/virtual-machines-shared-sap-high-availability-guide/3001-configuring-dns-servers-for-Azure-vnet.png
[sap-ha-guide-figure-3002]:./media/virtual-machines-shared-sap-high-availability-guide/3002-configuring-static-IP-address-for-network-card-of-each-vm.png
[sap-ha-guide-figure-3003]:./media/virtual-machines-shared-sap-high-availability-guide/3003-setup-static-ip-address-ilb-for-ascs-instance.png
[sap-ha-guide-figure-3004]:./media/virtual-machines-shared-sap-high-availability-guide/3004-default-ascs-scs-ilb-balancing-rules-for-azure-ilb.png
[sap-ha-guide-figure-3005]:./media/virtual-machines-shared-sap-high-availability-guide/3005-changing-ascs-scs-default-ilb-rules-for-azure-ilb.png
[sap-ha-guide-figure-3006]:./media/virtual-machines-shared-sap-high-availability-guide/3006-adding-vm-to-domain.png
[sap-ha-guide-figure-3007]:./media/virtual-machines-shared-sap-high-availability-guide/3007-config-wsfc-1.png
[sap-ha-guide-figure-3008]:./media/virtual-machines-shared-sap-high-availability-guide/3008-config-wsfc-2.png
[sap-ha-guide-figure-3009]:./media/virtual-machines-shared-sap-high-availability-guide/3009-config-wsfc-3.png
[sap-ha-guide-figure-3010]:./media/virtual-machines-shared-sap-high-availability-guide/3010-config-wsfc-4.png
[sap-ha-guide-figure-3011]:./media/virtual-machines-shared-sap-high-availability-guide/3011-config-wsfc-5.png
[sap-ha-guide-figure-3012]:./media/virtual-machines-shared-sap-high-availability-guide/3012-config-wsfc-6.png
[sap-ha-guide-figure-3013]:./media/virtual-machines-shared-sap-high-availability-guide/3013-config-wsfc-7.png
[sap-ha-guide-figure-3014]:./media/virtual-machines-shared-sap-high-availability-guide/3014-config-wsfc-8.png
[sap-ha-guide-figure-3015]:./media/virtual-machines-shared-sap-high-availability-guide/3015-config-wsfc-9.png
[sap-ha-guide-figure-3016]:./media/virtual-machines-shared-sap-high-availability-guide/3016-config-wsfc-10.png
[sap-ha-guide-figure-3017]:./media/virtual-machines-shared-sap-high-availability-guide/3017-config-wsfc-11.png
[sap-ha-guide-figure-3018]:./media/virtual-machines-shared-sap-high-availability-guide/3018-config-wsfc-12.png
[sap-ha-guide-figure-3019]:./media/virtual-machines-shared-sap-high-availability-guide/3019-assign-permissions-on-share-for-cluster-name-object.png
[sap-ha-guide-figure-3020]:./media/virtual-machines-shared-sap-high-availability-guide/3020-change-object-type-include-computer-objects.png
[sap-ha-guide-figure-3021]:./media/virtual-machines-shared-sap-high-availability-guide/3021-check-box-for-computer-objects.png
[sap-ha-guide-figure-3022]:./media/virtual-machines-shared-sap-high-availability-guide/3022-set-security-attributes-for-cluster-name-object-on-file-share-quorum.png
[sap-ha-guide-figure-3023]:./media/virtual-machines-shared-sap-high-availability-guide/3023-call-configure-cluster-quorum-setting-wizard.png
[sap-ha-guide-figure-3024]:./media/virtual-machines-shared-sap-high-availability-guide/3024-selection-screen-different-quorum-configurations.png
[sap-ha-guide-figure-3025]:./media/virtual-machines-shared-sap-high-availability-guide/3025-selection-screen-file-share-witness.png
[sap-ha-guide-figure-3026]:./media/virtual-machines-shared-sap-high-availability-guide/3026-define-file-share-location-for-witness-share.png
[sap-ha-guide-figure-3027]:./media/virtual-machines-shared-sap-high-availability-guide/3027-successful-reconfiguration-cluster-file-share-witness.png
[sap-ha-guide-figure-3028]:./media/virtual-machines-shared-sap-high-availability-guide/3028-install-dot-net-framework-35.png
[sap-ha-guide-figure-3029]:./media/virtual-machines-shared-sap-high-availability-guide/3029-install-dot-net-framework-35-progress.png
[sap-ha-guide-figure-3030]:./media/virtual-machines-shared-sap-high-availability-guide/3030-sios-installer.png
[sap-ha-guide-figure-3031]:./media/virtual-machines-shared-sap-high-availability-guide/3031-first-screen-sios-data-keeper-installation.png
[sap-ha-guide-figure-3032]:./media/virtual-machines-shared-sap-high-availability-guide/3032-data-keeper-informs-service-be-disabled.png
[sap-ha-guide-figure-3033]:./media/virtual-machines-shared-sap-high-availability-guide/3033-user-selection-sios-data-keeper.png
[sap-ha-guide-figure-3034]:./media/virtual-machines-shared-sap-high-availability-guide/3034-domain-user-sios-data-keeper.png
[sap-ha-guide-figure-3035]:./media/virtual-machines-shared-sap-high-availability-guide/3035-provide-sios-data-keeper-license.png
[sap-ha-guide-figure-3036]:./media/virtual-machines-shared-sap-high-availability-guide/3036-data-keeper-management-config-tool.png
[sap-ha-guide-figure-3037]:./media/virtual-machines-shared-sap-high-availability-guide/3037-tcp-ip-address-first-node-data-keeper.png
[sap-ha-guide-figure-3038]:./media/virtual-machines-shared-sap-high-availability-guide/3038-create-replication-sios-job.png
[sap-ha-guide-figure-3039]:./media/virtual-machines-shared-sap-high-availability-guide/3039-define-sios-replication-job-name.png
[sap-ha-guide-figure-3040]:./media/virtual-machines-shared-sap-high-availability-guide/3040-define-sios-source-node.png
[sap-ha-guide-figure-3041]:./media/virtual-machines-shared-sap-high-availability-guide/3041-define-sios-target-node.png
[sap-ha-guide-figure-3042]:./media/virtual-machines-shared-sap-high-availability-guide/3042-define-sios-synchronous-replication.png
[sap-ha-guide-figure-3043]:./media/virtual-machines-shared-sap-high-availability-guide/3043-enable-sios-replicated-volume-as-cluster-volume.png
[sap-ha-guide-figure-3044]:./media/virtual-machines-shared-sap-high-availability-guide/3044-data-keeper-synchronous-mirroring-for-SAP-gui.png
[sap-ha-guide-figure-3045]:./media/virtual-machines-shared-sap-high-availability-guide/3045-replicated-disk-by-data-keeper-in-wsfc.png
[sap-ha-guide-figure-3046]:./media/virtual-machines-shared-sap-high-availability-guide/3046-dns-entry-sap-ascs-virtual-name-ip.png
[sap-ha-guide-figure-3047]:./media/virtual-machines-shared-sap-high-availability-guide/3047-dns-manager.png
[sap-ha-guide-figure-3048]:./media/virtual-machines-shared-sap-high-availability-guide/3048-default-cluster-probe-port.png
[sap-ha-guide-figure-3049]:./media/virtual-machines-shared-sap-high-availability-guide/3049-cluster-probe-port-after.png
[sap-ha-guide-figure-3050]:./media/virtual-machines-shared-sap-high-availability-guide/3050-service-type-ers-delayed-automatic.png
[sap-ha-guide-figure-5000]:./media/virtual-machines-shared-sap-high-availability-guide/5000-wsfc-sap-sid-node-a.png
[sap-ha-guide-figure-5001]:./media/virtual-machines-shared-sap-high-availability-guide/5001-sios-replicating-local-volume.png
[sap-ha-guide-figure-5002]:./media/virtual-machines-shared-sap-high-availability-guide/5002-wsfc-sap-sid-node-b.png
[sap-ha-guide-figure-5003]:./media/virtual-machines-shared-sap-high-availability-guide/5003-sios-replicating-local-volume-b-to-a.png

[sap-ha-guide-figure-6003]:./media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png


[sap-ha-guide-figure-8001]:./media/virtual-machines-shared-sap-high-availability-guide/8001.png
[sap-ha-guide-figure-8002]:./media/virtual-machines-shared-sap-high-availability-guide/8002.png
[sap-ha-guide-figure-8003]:./media/virtual-machines-shared-sap-high-availability-guide/8003.png
[sap-ha-guide-figure-8004]:./media/virtual-machines-shared-sap-high-availability-guide/8004.png
[sap-ha-guide-figure-8005]:./media/virtual-machines-shared-sap-high-availability-guide/8005.png
[sap-ha-guide-figure-8006]:./media/virtual-machines-shared-sap-high-availability-guide/8006.png
[sap-ha-guide-figure-8007]:./media/virtual-machines-shared-sap-high-availability-guide/8007.png
[sap-ha-guide-figure-8008]:./media/virtual-machines-shared-sap-high-availability-guide/8008.png
[sap-ha-guide-figure-8009]:./media/virtual-machines-shared-sap-high-availability-guide/8009.png
[sap-ha-guide-figure-8010]:./media/virtual-machines-shared-sap-high-availability-guide/8010.png
[sap-ha-guide-figure-8011]:./media/virtual-machines-shared-sap-high-availability-guide/8011.png
[sap-ha-guide-figure-8012]:./media/virtual-machines-shared-sap-high-availability-guide/8012.png
[sap-ha-guide-figure-8013]:./media/virtual-machines-shared-sap-high-availability-guide/8013.png
[sap-ha-guide-figure-8014]:./media/virtual-machines-shared-sap-high-availability-guide/8014.png
[sap-ha-guide-figure-8015]:./media/virtual-machines-shared-sap-high-availability-guide/8015.png
[sap-ha-guide-figure-8016]:./media/virtual-machines-shared-sap-high-availability-guide/8016.png
[sap-ha-guide-figure-8017]:./media/virtual-machines-shared-sap-high-availability-guide/8017.png
[sap-ha-guide-figure-8018]:./media/virtual-machines-shared-sap-high-availability-guide/8018.png
[sap-ha-guide-figure-8019]:./media/virtual-machines-shared-sap-high-availability-guide/8019.png
[sap-ha-guide-figure-8020]:./media/virtual-machines-shared-sap-high-availability-guide/8020.png
[sap-ha-guide-figure-8021]:./media/virtual-machines-shared-sap-high-availability-guide/8021.png
[sap-ha-guide-figure-8022]:./media/virtual-machines-shared-sap-high-availability-guide/8022.png
[sap-ha-guide-figure-8023]:./media/virtual-machines-shared-sap-high-availability-guide/8023.png
[sap-ha-guide-figure-8024]:./media/virtual-machines-shared-sap-high-availability-guide/8024.png
[sap-ha-guide-figure-8025]:./media/virtual-machines-shared-sap-high-availability-guide/8025.png


[sap-templates-3-tier-multisid-xscs-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs%2Fazuredeploy.json
[sap-templates-3-tier-multisid-xscs-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps-md%2Fazuredeploy.json

[virtual-machines-azure-resource-manager-architecture-benefits-arm]:../../../azure-resource-manager/management/overview.md#the-benefits-of-using-resource-manager

[virtual-machines-manage-availability]:../../virtual-machines-windows-manage-availability.md

[1869038]:https://launchpad.support.sap.com/#/notes/1869038 
