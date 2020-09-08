---
title: Infraestructura de Azure para ASCS/SCS de SAP con WSFC y disco compartido| Microsoft Docs
description: Aprenda cómo preparar la infraestructura de Azure para alta disponibilidad de SAP con un clúster de conmutación por error de Windows y un disco compartido para una instancia de ASCS/SCS de SAP.
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: ec976257-396b-42a0-8ea1-01c97f820fa6
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/25/2020
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 8f389581d8fbeb912507b303c46109dd08fcab8d
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88871523"
---
# <a name="prepare-the-azure-infrastructure-for-sap-ha-by-using-a-windows-failover-cluster-and-shared-disk-for-sap-ascsscs"></a>Preparación de la infraestructura de Azure para alta disponibilidad de SAP con un clúster de conmutación por error de Windows y un disco compartido para ASCS/SCS de SAP

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[sap-installation-guides]:http://service.sap.com/instguides
[tuning-failover-cluster-network-thresholds]:https://techcommunity.microsoft.com/t5/Failover-Clustering/Tuning-Failover-Cluster-Network-Thresholds/ba-p/371834

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
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-ha-guide-9.1.1]:high-availability-guide.md#a97ad604-9094-44fe-a364-f89cb39bf097
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f


[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-shared-disk-vpn]:sap-high-availability-infrastructure-wsfc-shared-disk.md#c87a8d3f-b1dc-4d2f-b23c-da4b72977489
[sap-high-availability-infrastructure-wsfc-shared-disk-change-def-ilb]:sap-high-availability-infrastructure-wsfc-shared-disk.md#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-high-availability-infrastructure-wsfc-shared-disk-setup-wsfc]:sap-high-availability-infrastructure-wsfc-shared-disk.md#0d67f090-7928-43e0-8772-5ccbf8f59aab
[sap-high-availability-infrastructure-wsfc-shared-disk-collect-cluster-config]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5c8e5482-841e-45e1-a89d-a05c0907c868
[sap-high-availability-infrastructure-wsfc-shared-disk-add-dot-net]:sap-high-availability-infrastructure-wsfc-shared-disk.md#1c2788c3-3648-4e82-9e0d-e058e475e2a3
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios-both-nodes]:sap-high-availability-infrastructure-wsfc-shared-disk.md#dd41d5a2-8083-415b-9878-839652812102
[sap-high-availability-infrastructure-wsfc-shared-disk-setup-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#d9c1fc8e-8710-4dff-bec2-1f535db7b006

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

[sap-templates-3-tier-multisid-xscs-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs%2Fazuredeploy.json
[sap-templates-3-tier-multisid-xscs-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps-md%2Fazuredeploy.json

[virtual-machines-azure-resource-manager-architecture-benefits-arm]:../../../azure-resource-manager/management/overview.md#the-benefits-of-using-resource-manager

[virtual-machines-manage-availability]:../../virtual-machines-windows-manage-availability.md


> ![SO Windows][Logo_Windows] Windows


Este artículo describe los pasos que debe seguir para preparar la infraestructura de Azure para instalar y configurar una instancia de ASCS/SCS de SAP de alta disponibilidad en un clúster de conmutación por error de Windows mediante el uso de un *disco compartido de clúster* como una opción de agrupación en clústeres de una instancia de ASCS de SAP.
En la documentación se presentan dos alternativas para el *disco compartido de clúster*:

- [Discos compartidos de Azure](https://docs.microsoft.com/azure/virtual-machines/windows/disks-shared)
- Uso de [SIOS DataKeeper Cluster Edition](https://us.sios.com/products/datakeeper-cluster/) para crear un almacenamiento reflejado, que simulará el disco compartido en clúster 

La configuración presentada se basa en [grupos con ubicación por proximidad de Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-proximity-placement-scenarios) para lograr una latencia de red óptima para cargas de trabajo de SAP. La documentación no abarca el nivel de base de datos.  

> [!NOTE]
> De hecho, los grupos con ubicación por proximidad son un requisito previo para el uso de discos compartidos de Azure.
 

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar la instalación, consulte este artículo:

* [Guía de arquitectura: Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un disco compartido de clúster][sap-high-availability-guide-wsfc-shared-disk]

## <a name="create-the-ascs-vms"></a>Creación de las máquinas virtuales de ASCS

En el caso del clúster de ASCS/SCS de SAP, implemente dos máquinas virtuales en el conjunto de disponibilidad de Azure. Implemente las máquinas virtuales en el mismo grupo con ubicación por proximidad. Una vez implementadas las máquinas virtuales:  
- Cree el equilibrador de carga interno de Azure para la instancia de ASCS/SCS de SAP. 
- Agregue máquinas virtuales de Windows al dominio de AD.

Los nombres de host y las direcciones IP para el escenario presentado son:

| Rol de nombre de host | Nombre de host | Dirección IP estática | Conjunto de disponibilidad | Grupo con ubicación por proximidad |
| --- | --- | --- |---| ---|
| Primer clúster ASCS/SCS de nodo de clúster |pr1-ascs-10 |10.0.0.4 |pr1-ascs-avset |PR1PPG |
| Segundo clúster ASCS/SCS de nodo de clúster |pr1-ascs-11 |10.0.0.5 |pr1-ascs-avset |PR1PPG |
| Nombre de red del clúster | pr1clust |10.0.0.42 (**solo** para el clúster de Win 2016) | N/D | N/D |
| Nombre de red del clúster de ASCS | pr1-ascscl |10.0.0.43 | N/D | N/D |
| Nombre de red del clúster de ERS (**solo** para ERS2) | pr1-erscl |10.0.0.44 | N/D | N/D |


## <a name="create-azure-internal-load-balancer"></a><a name="fe0bd8b5-2b43-45e3-8295-80bee5415716"></a> Creación de un equilibrador de carga interno de Azure

ASCS de SAP, SCS de SAP y el nuevo ERS2 de SAP usan direcciones IP virtuales y un nombre de host virtual. En Azure, se requiere un [equilibrador de carga](https://docs.microsoft.com/azure/load-balancer/load-balancer-overview) para usar una dirección IP virtual. Se recomienda encarecidamente usar [Standard Load Balancer](https://docs.microsoft.com/azure/load-balancer/quickstart-load-balancer-standard-public-portal). 


En la lista siguiente se muestra la configuración del equilibrador de carga (A)SCS/ERS. La configuración de ASCS de SAP y ERS2 se realiza en el mismo equilibrador de carga de Azure.  

**(A)SCS**
- Configuración de front-end
    - Dirección IP estática de ASCS/SCS **10.0.0.43**
- Configuración de back-end  
    Agregue todas las máquinas virtuales que deben formar parte del clúster de (A)SCS/ERS. En este ejemplo, las máquinas virtuales **pr1-ascs-10** y **pr1-ascs-11**.
- Puerto de sondeo
    - Puerto 620**nr** Deje la opción predeterminada para Protocolo (TCP), Intervalo (5), Umbral incorrecto (2)
- Reglas de equilibrio de carga.
    - Si usa Standard Load Balancer, seleccione Puertos HA
    - Si usa Basic Load Balancer, cree reglas de equilibrio de carga para los puertos siguientes
        - 32**nr** TCP
        - 36**nr** TCP
        - 39**nr** TCP
        - 81**nr** TCP
        - 5**nr**13 TCP
        - 5**nr**14 TCP
        - 5**nr**16 TCP

    - Asegúrese de que el tiempo de expiración de inactividad (minutos) esté establecido en el valor máximo de 30, y que la dirección IP flotante (Direct Server Return) esté habilitada.

**ERS2**

Como Enqueue Replication Server 2 (ERS2) también es un clúster, la dirección IP virtual de ERS2 también debe estar configurada en Azure ILB además de la IP de ASCS/SCS de SAP anterior. Esta sección solo se aplica si se usa la arquitectura de Enqueue Replication Server 2.  
- Segunda configuración de front-end
    - Dirección IP de ERS2 de SAP estática **10.0.0.44**

- Configuración de back-end  
  Las máquinas virtuales ya se han agregado al grupo de back-end de ILB.  

- Segundo puerto de sondeo
    - Puerto 621**nr**  
    Deje la opción predeterminada para Protocolo (TCP), Intervalo (5), Umbral incorrecto (2)

- Segundas reglas de equilibrio de carga
    - Si usa Standard Load Balancer, seleccione Puertos HA
    - Si usa Basic Load Balancer, cree reglas de equilibrio de carga para los puertos siguientes
        - 32**nr** TCP
        - 33**nr** TCP
        - 5**nr**13 TCP
        - 5**nr**14 TCP
        - 5**nr**16 TCP

    - Asegúrese de que el tiempo de expiración de inactividad (minutos) esté establecido en el valor máximo de 30, por ejemplo, y que la dirección IP flotante (Direct Server Return) esté habilitada.


> [!TIP]
> Con la [plantilla de Azure Resource Manager para WSFC para la instancia de ASCS/SCS de SAP con un disco compartido de Azure](https://github.com/robotechredmond/301-shared-disk-sap), puede automatizar la preparación de la infraestructura mediante el uso del disco compartido de Azure para un SID de SAP con ERS1.  
> La plantilla de Resource Manager de Azure creará dos máquinas virtuales Windows 2019 o 2016, creará un disco compartido de Azure y se conectará a las máquinas virtuales. También se creará y configurará el equilibrador de carga interno de Azure. Para obtener más información, consulte la plantilla de Resource Manager. 

## <a name="add-registry-entries-on-both-cluster-nodes-of-the-ascsscs-instance"></a><a name="661035b2-4d0f-4d31-86f8-dc0a50d78158"></a> Incorporación de entradas del registro en ambos nodos de la instancia de ASCS/SCS

Azure Load Balancer pueden cerrar conexiones, si las conexiones están inactivas durante un período y superan el tiempo de espera de inactividad. Los proceso de trabajo de SAP abren conexiones con el proceso de puesta en cola de SAP tan pronto como se deba enviar la primera solicitud para poner en cola y para quitar de la cola. Para evitar interrumpir estas conexiones, cambie los valores de KeepAliveTime y KeepAliveInterval de TCP/IP en ambos nodos del clúster. Si usa ERS1, también es necesario agregar parámetros de perfil de SAP, como se describe más adelante en este artículo.
Se deben cambiar las siguientes entradas del registro en ambos nodos del clúster:

- KeepAliveTime
- KeepAliveInterval

| Ruta de acceso| Nombre de la variable | Tipo de variable  | Value | Documentación |
| --- | --- | --- |---| ---|
| HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |KeepAliveTime |REG_DWORD (Decimal) |120000 |[KeepAliveTime](https://technet.microsoft.com/library/cc957549.aspx) |
| HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |KeepAliveInterval |REG_DWORD (Decimal) |120000 |[KeepAliveInterval](https://technet.microsoft.com/library/cc957548.aspx) |


Para aplicar los cambios, reinicie ambos nodos del clúster.
  
## <a name="add-the-windows-vms-to-the-domain"></a><a name="e69e9a34-4601-47a3-a41c-d2e11c626c0c"></a> Incorporación de las máquinas virtuales de Windows al dominio
Después de asignar direcciones IP estáticas a las máquinas virtuales, agregue las máquinas virtuales al dominio. 

## <a name="install-and-configure--windows-failover-cluster"></a><a name="0d67f090-7928-43e0-8772-5ccbf8f59aab"></a> Instalación y configuración de un clúster de conmutación por error de Windows 

### <a name="install-the-windows-failover-cluster-feature"></a>Instalación de la característica del clúster de conmutación por error de Windows

Ejecute este comando en uno de los nodos del clúster:

   ```powershell
    # Hostnames of the Win cluster for SAP ASCS/SCS
    $SAPSID = "PR1"
    $ClusterNodes = ("pr1-ascs-10","pr1-ascs-11")
    $ClusterName = $SAPSID.ToLower() + "clust"
    
    # Install Windows features.
    # After the feature installs, manually reboot both nodes
    Invoke-Command $ClusterNodes {Install-WindowsFeature Failover-Clustering, FS-FileServer -IncludeAllSubFeature -IncludeManagementTools }
   ```

Una vez completada la instalación de la característica, reinicie ambos nodos del clúster.  

### <a name="test-and-configure-windows-failover-cluster"></a>Prueba y configuración de un clúster de conmutación por error de Windows 

En Windows 2019, el clúster reconocerá automáticamente que se ejecuta en Azure y, como opción predeterminada para la dirección IP de administración de clústeres, usará el nombre de red distribuida. Por lo tanto, usará cualquiera de las direcciones IP locales de los nodos del clúster. Como resultado, no es necesario un nombre de red dedicado (virtual) para el clúster, y no es necesario configurar esta dirección IP en el equilibrador de carga interno de Azure.

Para obtener más información, consulte [Nuevas características de clústeres de conmutación por error de Windows Server 2019](https://techcommunity.microsoft.com/t5/failover-clustering/windows-server-2019-failover-clustering-new-features/ba-p/544029) Ejecute este comando en uno de los nodos del clúster:

   ```powershell
    # Hostnames of the Win cluster for SAP ASCS/SCS
    $SAPSID = "PR1"
    $ClusterNodes = ("pr1-ascs-10","pr1-ascs-11")
    $ClusterName = $SAPSID.ToLower() + "clust"
    
    # IP adress for cluster network name is needed ONLY on Windows Server 2016 cluster
    $ClusterStaticIPAddress = "10.0.0.42"
        
    # Test cluster
    Test-Cluster –Node $ClusterNodes -Verbose
    
    $ComputerInfo = Get-ComputerInfo
    
    $WindowsVersion = $ComputerInfo.WindowsProductName
    
    if($WindowsVersion -eq "Windows Server 2019 Datacenter"){
        write-host "Configuring Windows Failover Cluster on Windows Server 2019 Datacenter..."
        New-Cluster –Name $ClusterName –Node  $ClusterNodes -Verbose
    }elseif($WindowsVersion -eq "Windows Server 2016 Datacenter"){
        write-host "Configuring Windows Failover Cluster on Windows Server 2016 Datacenter..."
        New-Cluster –Name $ClusterName –Node  $ClusterNodes –StaticAddress $ClusterStaticIPAddress -Verbose 
    }else{
        Write-Error "Not supported Windows version!"
    }
   ```

### <a name="configure-cluster-cloud-quorum"></a>Configuración del cuórum de nube del clúster
A medida que use Windows Server 2016 o 2019, se recomienda configurar el [testigo en la nube de Azure](https://docs.microsoft.com/windows-server/failover-clustering/deploy-cloud-witness) como cuórum del clúster.

Ejecute este comando en uno de los nodos del clúster:

   ```powershell
    $AzureStorageAccountName = "cloudquorumwitness"
    Set-ClusterQuorum –CloudWitness –AccountName $AzureStorageAccountName -AccessKey <YourAzureStorageAccessKey> -Verbose
   ```

### <a name="tuning-the-windows-failover-cluster-thresholds"></a>Ajuste de los umbrales del clúster de conmutación por error de Windows
 
Después de instalar correctamente el clúster de conmutación por error de Windows, debe ajustar algunos umbrales para que sea adecuado para los clústeres implementados en Azure. Los parámetros que se van a cambiar se documentan en [Ajuste de los umbrales de la red en clúster de conmutación por error](https://techcommunity.microsoft.com/t5/Failover-Clustering/Tuning-Failover-Cluster-Network-Thresholds/ba-p/371834). Suponiendo que las dos máquinas virtuales que forman la configuración del clúster de Windows para ASCS/SCS están en la misma subred, los parámetros siguientes deben cambiarse a estos valores:
- SameSubNetDelay = 2000
- SameSubNetThreshold = 15
- RoutingHistoryLength = 30

Esta configuración se ha probado con clientes y ofrece un buen compromiso. Es lo suficientemente resistente y también proporciona una conmutación por error que es lo suficientemente rápida en condiciones de error real en cargas de trabajo de SAP o en el error de una máquina virtual.  

## <a name="configure-azure-shared-disk"></a>Configuración de un disco compartido de Azure
Esta sección solo es aplicable si usa el disco compartido de Azure. 

### <a name="create-and-attach-azure-shared-disk-with-powershell"></a>Creación y conexión de un disco compartido de Azure con PowerShell
Ejecute este comando en uno de los nodos del clúster. Tendrá que ajustar los valores para el grupo de recursos, la región de Azure, SAPSID, etc.  

   ```powershell
    #############################
    # Create Azure Shared Disk
    #############################
    
    $ResourceGroupName = "MyResourceGroup"
    $location = "MyAzureRegion"
    $SAPSID = "PR1"
    
    $DiskSizeInGB = 512
    $DiskName = "$($SAPSID)ASCSSharedDisk"
    
    # With parameter '-MaxSharesCount', we define the maximum number of cluster nodes to attach the shared disk
    $NumberOfWindowsClusterNodes = 2
            
    $diskConfig = New-AzDiskConfig -Location $location -SkuName Premium_LRS  -CreateOption Empty  -DiskSizeGB $DiskSizeInGB -MaxSharesCount $NumberOfWindowsClusterNodes
    $dataDisk = New-AzDisk -ResourceGroupName $ResourceGroupName -DiskName $DiskName -Disk $diskConfig
    
    ##################################
    ## Attach the disk to cluster VMs
    ##################################
    # ASCS Cluster VM1
    $ASCSClusterVM1 = "$SAPSID-ascs-10"
    
    # ASCS Cluster VM2
    $ASCSClusterVM2 = "$SAPSID-ascs-11"
    
    # Add the Azure Shared Disk to Cluster Node 1
    $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $ASCSClusterVM1 
    $vm = Add-AzVMDataDisk -VM $vm -Name $DiskName -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun 0
    Update-AzVm -VM $vm -ResourceGroupName $ResourceGroupName -Verbose
    
    # Add the Azure Shared Disk to Cluster Node 2
    $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $ASCSClusterVM2
    $vm = Add-AzVMDataDisk -VM $vm -Name $DiskName -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun 0
    Update-AzVm -VM $vm -ResourceGroupName $ResourceGroupName -Verbose
   ```

### <a name="format-the-shared-disk-with-powershell"></a>Formato del disco compartido con PowerShell
1. Obtenga el número de disco. Ejecute los comandos de PowerShell en uno de los nodos del clúster:

   ```powershell
    Get-Disk | Where-Object PartitionStyle -Eq "RAW"  | Format-Table -AutoSize 
    # Example output
    # Number Friendly Name     Serial Number HealthStatus OperationalStatus Total Size Partition Style
    # ------ -------------     ------------- ------------ ----------------- ---------- ---------------
    # 2      Msft Virtual Disk               Healthy      Online                512 GB RAW            

   ```
2. Aplique formato al disco. En este ejemplo, el número de disco es 2. 

   ```powershell
    # Format SAP ASCS Disk number '2', with drive letter 'S'
    $SAPSID = "PR1"
    $DiskNumber = 2
    $DriveLetter = "S"
    $DiskLabel = "$SAPSID" + "SAP"
    
    Get-Disk -Number $DiskNumber | Where-Object PartitionStyle -Eq "RAW" | Initialize-Disk -PartitionStyle GPT -PassThru |  New-Partition -DriveLetter $DriveLetter -UseMaximumSize | Format-Volume  -FileSystem ReFS -NewFileSystemLabel $DiskLabel -Force -Verbose
    # Example outout
    # DriveLetter FileSystemLabel FileSystem DriveType HealthStatus OperationalStatus SizeRemaining      Size
    # ----------- --------------- ---------- --------- ------------ ----------------- -------------      ----
    # S           PR1SAP          ReFS       Fixed     Healthy      OK                    504.98 GB 511.81 GB
   ```

3. Compruebe que el disco esté ahora visible como disco de clúster.  
   ```powershell
    # List all disks
    Get-ClusterAvailableDisk -All
    # Example output
    # Cluster    : pr1clust
    # Id         : 88ff1d94-0cf1-4c70-89ae-cbbb2826a484
    # Name       : Cluster Disk 1
    # Number     : 2
    # Size       : 549755813888
    # Partitions : {\\?\GLOBALROOT\Device\Harddisk2\Partition2\}
   ```
4. Registre el disco en el clúster.  
   ```powershell
    # Add the disk to cluster 
    Get-ClusterAvailableDisk -All | Add-ClusterDisk
    # Example output     
    # Name           State  OwnerGroup        ResourceType 
    # ----           -----  ----------        ------------ 
    # Cluster Disk 1 Online Available Storage Physical Disk
   ```

## <a name="sios-datakeeper-cluster-edition-for-the-sap-ascsscs-cluster-share-disk"></a><a name="5c8e5482-841e-45e1-a89d-a05c0907c868"></a> SIOS DataKeeper Cluster Edition para un disco compartido de clúster de ASCS/SCS de SAP
Esta sección solo es aplicable si usa el software de terceros SIOS DataKeeper Cluster Edition para crear un almacenamiento reflejado que simula el disco compartido de clúster.  

Ahora tiene una configuración de clústeres de conmutación por error de Windows Server activa en Azure. Para instalar una instancia de ASCS/SCS de SAP, necesita un recurso de disco compartido. SIOS DataKeeper Cluster Edition es una solución de terceros que puede usar para crear recursos de disco compartido.  

Para instalar SIOS DataKeeper Cluster Edition en un disco compartido de clúster de ASCS/SCS de SAP, siga estos pasos:
- Agregue Microsoft .NET Framework, si se necesita. Consulte la [documentación de SIOS] ((https://us.sios.com/products/datakeeper-cluster/) para ver los requisitos de .NET Framework más actualizados. 
-  Instalación de SIOS DataKeeper
- Configuración de SIOS DataKeeper

### <a name="install-sios-datakeeper"></a> Instalación de SIOS DataKeeper
Instale SIOS DataKeeper Cluster Edition en cada nodo del clúster. Con SIOS DataKeeper, para crear el almacenamiento compartido virtual, cree un reflejo sincronizado y, luego, simule el almacenamiento compartido de clúster.

Antes de instalar el software de SIOS, cree el usuario de dominio DataKeeperSvc.

> [!NOTE]
> Agregue el usuario DataKeeperSvc al grupo de administradores locales en ambos nodos del clúster.
>

1. Instale el software SIOS en ambos nodos del clúster.

   ![Instalador de SIOS][sap-ha-guide-figure-3030]

   ![Figura 31: Primera página de la instalación de SIOS DataKeeper][sap-ha-guide-figure-3031]

   _Primera página de la instalación de SIOS DataKeeper_

2. En el cuadro de diálogo, seleccione **Sí**.

   ![Figura 32: DataKeeper le informa que se deshabilitará un servicio][sap-ha-guide-figure-3032]

   _Información de DataKeeper de la próxima deshabilitación de un servicio_

3. En el cuadro de diálogo, se recomienda seleccionar **Cuenta de dominio o de servidor**.

   ![Figura 33: Selección del usuario para SIOS DataKeeper][sap-ha-guide-figure-3033]

   _Selección del usuario para SIOS DataKeeper_

4. Escriba el nombre de usuario y la contraseña de la cuenta de dominio que creó para SIOS DataKeeper.

   ![Figura 34: Introducción del nombre de usuario y contraseña de dominio para la instalación de SIOS DataKeeper][sap-ha-guide-figure-3034]

   _Introducción del nombre de usuario y contraseña de dominio para la instalación de SIOS DataKeeper_

5. Instale la clave de licencia para la instancia de SIOS DataKeeper, tal como aparece en la figura 35.

   ![Figura 35: Especificación de la licencia de SIOS DataKeeper][sap-ha-guide-figure-3035]

   _Especificación de la clave de licencia de SIOS DataKeeper_

6. Cuando se le solicite, reinicie la máquina virtual.

### <a name="configure-sios-datakeeper"></a>Configuración de SIOS DataKeeper
Después de instalar SIOS DataKeeper en ambos nodos, inicie la configuración. El objetivo de la configuración es conseguir la replicación sincrónica de datos entre los discos adicionales conectados a cada una de las máquinas virtuales.

1. Inicie la herramienta de configuración y administración de DataKeeper y, luego, seleccione **Servidor de conexión**.

   ![Figura 36: Herramienta de configuración y administración de SIOS DataKeeper][sap-ha-guide-figure-3036]

   _Herramienta de configuración y administración de SIOS DataKeeper_

2. Escriba el nombre o la dirección TCP/IP del primer nodo al que se debe conectar la herramienta de configuración y administración y, en un segundo paso, el segundo nodo.

   ![Figura 37: Inserte el nombre o la dirección TCP/IP del primer nodo al que se debe conectar la herramienta de configuración y administración y, en un segundo paso, el segundo nodo][sap-ha-guide-figure-3037]

   _Inserción del nombre o la dirección TCP/IP del primer nodo al que se debe conectar la herramienta de configuración y administración y, en un segundo paso, el segundo nodo_.

3. Cree el trabajo de replicación entre los dos nodos.

   ![Figura 38: Creación de un trabajo de replicación][sap-ha-guide-figure-3038]

   _Creación de un trabajo de replicación_

   Un asistente le guía por el proceso de crear un trabajo de replicación.

4. Defina el nombre del trabajo de replicación.

   ![Figura 39: Definición del nombre del trabajo de replicación][sap-ha-guide-figure-3039]

   _Definición del nombre del trabajo de replicación_

   ![Figura 40: Definición de los datos básicos para el nodo que debe ser el nodo de origen actual][sap-ha-guide-figure-3040]

   _Definición de los datos básicos para el nodo que debe ser el nodo de origen actual_

5. Defina el nombre, la dirección TCP/IP y el volumen de disco del nodo de destino.

   ![Figura 41: Definición del nombre, la dirección TCP/IP y el volumen de disco del nodo de destino actual][sap-ha-guide-figure-3041]

   _Definición del nombre, la dirección TCP/IP y el volumen de disco del nodo de destino actual_

6. Defina los algoritmos de compresión. En el ejemplo, se recomienda comprimir el flujo de replicación. Especialmente en situaciones de resincronización, la compresión del flujo de replicación reduce considerablemente el tiempo que se tarda en resincronizar. La compresión usa los recursos de CPU y RAM de una máquina virtual. A medida que aumenta la tasa de compresión, también aumenta el volumen de los recursos de CPU usados. Puede ajustar esta configuración más adelante.

7. Otra configuración que debe comprobar es si la replicación se ejecuta de forma sincrónica o asincrónica. Cuando proteja configuraciones de ASCS/SCS de SAP, debe usar la replicación sincrónica.  

   ![Figura 42: Definición de los detalles de la replicación][sap-ha-guide-figure-3042]

   _Definición de los detalles de la replicación_

8. Defina si el volumen que el trabajo de replicación replica se debe representar en una configuración de clúster de conmutación por error de Windows Server como disco compartido. Para la configuración de ASCS/SCS de SAP, seleccione **Sí** de manera que el clúster de Windows vea el volumen replicado como un disco compartido que puede usar como volumen de clúster.

   ![Figura 43: Selección de Sí para establecer el volumen replicado como volumen de clúster][sap-ha-guide-figure-3043]

   _Selección de **Sí** para establecer el volumen replicado como volumen de clúster_

   Una vez creado el volumen, la herramienta de configuración y administración de DataKeeper muestra que el trabajo de replicación está activo.

   ![Figura 44: El reflejo sincrónico de DataKeeper para el disco compartido de ASCS/SCS de SAP está activo][sap-ha-guide-figure-3044]

   _El reflejo sincrónico de DataKeeper para el disco compartido de ASCS/SCS de SAP está activo_

   Ahora, el Administrador de clústeres de conmutación por error muestra el disco como un disco de DataKeeper, tal como aparece en la figura 45:

   ![Figura 45: El Administrador de clústeres de conmutación por error muestra el disco que DataKeeper replicó][sap-ha-guide-figure-3045]

   _El Administrador de clústeres de conmutación por error muestra el disco que DataKeeper replicó_


## <a name="next-steps"></a>Pasos siguientes

* [Instalación de alta disponibilidad para SAP NetWeaver con un clúster de conmutación por error de Windows y un disco compartido para una instancia de ASCS/SCS de SAP][sap-high-availability-installation-wsfc-shared-disk]
