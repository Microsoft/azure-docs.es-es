---
title: Arquitectura y escenarios de alta disponibilidad de máquinas virtuales de Azure para SAP NetWeaver | Microsoft Docs
description: Escenarios y arquitectura de alta disponibilidad para SAP NetWeaver en Azure Virtual Machines
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 887caaec-02ba-4711-bd4d-204a7d16b32b
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 02/26/2020
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a4ab403ebafbf078accd2ee2256c0c5bb69548e9
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288266"
---
# <a name="high-availability-architecture-and-scenarios-for-sap-netweaver"></a>Escenarios y arquitectura de alta disponibilidad para SAP NetWeaver

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png


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
[sap-ascs-ha-multi-sid-wsfc-file-share]:sap-ascs-ha-multi-sid-wsfc-file-share.md
[sap-ascs-ha-multi-sid-wsfc-shared-disk]:sap-ascs-ha-multi-sid-wsfc-shared-disk.md
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md
[sap-suse-ascs-ha-anf]:high-availability-guide-suse-netapp-files.md
[sap-higher-availability]:sap-higher-availability-architecture-scenarios.md

[planning-guide]:planning-guide.md  
[planning-guide-1.2]:planning-guide.md#e55d1e22-c2c8-460b-9897-64622a34fdff
[planning-guide-11]:planning-guide.md#7cf991a1-badd-40a9-944e-7baae842a058
[planning-guide-11.4.1]:planning-guide.md#5d9d36f9-9058-435d-8367-5ad05f00de77
[planning-guide-11.5]:planning-guide.md#4e165b58-74ca-474f-a7f4-5e695a93204f
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10
[planning-guide-3.1]:planning-guide.md#be80d1b9-a463-4845-bd35-f4cebdb5424a
[planning-guide-3.2.1]:planning-guide.md#df49dc09-141b-4f34-a4a2-990913b30358
[planning-guide-3.2.2]:planning-guide.md#fc1ac8b2-e54a-487c-8581-d3cc6625e560
[planning-guide-3.2.3]:planning-guide.md#18810088-f9be-4c97-958a-27996255c665
[planning-guide-3.2]:planning-guide.md#8d8ad4b8-6093-4b91-ac36-ea56d80dbf77
[planning-guide-3.3.2]:planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92
[planning-guide-5.1.1]:planning-guide.md#4d175f1b-7353-4137-9d2f-817683c26e53
[planning-guide-5.1.2]:planning-guide.md#e18f7839-c0e2-4385-b1e6-4538453a285c
[planning-guide-5.2.1]:planning-guide.md#1b287330-944b-495d-9ea7-94b83aff73ef
[planning-guide-5.2.2]:planning-guide.md#57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3
[planning-guide-5.2]:planning-guide.md#6ffb9f41-a292-40bf-9e70-8204448559e7
[planning-guide-5.3.1]:planning-guide.md#6e835de8-40b1-4b71-9f18-d45b20959b79
[planning-guide-5.3.2]:planning-guide.md#a43e40e6-1acc-4633-9816-8f095d5a7b6a
[planning-guide-5.4.2]:planning-guide.md#9789b076-2011-4afa-b2fe-b07a8aba58a1
[planning-guide-5.5.1]:planning-guide.md#4efec401-91e0-40c0-8e64-f2dceadff646
[planning-guide-5.5.3]:planning-guide.md#17e0d543-7e8c-4160-a7da-dd7117a1ad9d
[planning-guide-7.1]:planning-guide.md#3e9c3690-da67-421a-bc3f-12c520d99a30
[planning-guide-7]:planning-guide.md#96a77628-a05e-475d-9df3-fb82217e8f14
[planning-guide-9.1]:planning-guide.md#6f0a47f3-a289-4090-a053-2521618a28c3
[planning-guide-azure-premium-storage]:planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92

[virtual-machines-windows-portal-sql-alwayson-availability-groups-manual]:../../windows/sql/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual.md
[virtual-machines-windows-portal-sql-alwayson-int-listener]:../../../azure-sql/virtual-machines/windows/availability-group-load-balancer-portal-configure.md

[sap-ha-bc-virtual-env-hyperv-vmware-white-paper]:https://scn.sap.com/docs/DOC-44415
[sap-ha-partner-information]:https://scn.sap.com/docs/DOC-8541
[azure-sla]:https://azure.microsoft.com/support/legal/sla/
[azure-virtual-machines-manage-availability]:../../windows/manage-availability.md
[azure-storage-redundancy]:https://azure.microsoft.com/documentation/articles/storage-redundancy/
[azure-storage-managed-disks-overview]:https://docs.microsoft.com/azure/storage/storage-managed-disks-overview

[planning-guide-figure-100]:media/virtual-machines-shared-sap-planning-guide/100-single-vm-in-azure.png
[planning-guide-figure-1300]:media/virtual-machines-shared-sap-planning-guide/1300-ref-config-iaas-for-sap.png
[planning-guide-figure-1400]:media/virtual-machines-shared-sap-planning-guide/1400-attach-detach-disks.png
[planning-guide-figure-1600]:media/virtual-machines-shared-sap-planning-guide/1600-firewall-port-rule.png
[planning-guide-figure-1700]:media/virtual-machines-shared-sap-planning-guide/1700-single-vm-demo.png
[planning-guide-figure-1900]:media/virtual-machines-shared-sap-planning-guide/1900-vm-set-vnet.png
[planning-guide-figure-200]:media/virtual-machines-shared-sap-planning-guide/200-multiple-vms-in-azure.png
[planning-guide-figure-2100]:media/virtual-machines-shared-sap-planning-guide/2100-s2s.png
[planning-guide-figure-2200]:media/virtual-machines-shared-sap-planning-guide/2200-network-printing.png
[planning-guide-figure-2300]:media/virtual-machines-shared-sap-planning-guide/2300-sapgui-stms.png
[planning-guide-figure-2400]:media/virtual-machines-shared-sap-planning-guide/2400-vm-extension-overview.png
[planning-guide-figure-2500]:media/virtual-machines-shared-sap-planning-guide/planning-monitoring-overview-2502.png
[planning-guide-figure-2600]:media/virtual-machines-shared-sap-planning-guide/2600-sap-router-connection.png
[planning-guide-figure-2700]:media/virtual-machines-shared-sap-planning-guide/2700-exposed-sap-portal.png
[planning-guide-figure-2800]:media/virtual-machines-shared-sap-planning-guide/2800-endpoint-config.png
[planning-guide-figure-2900]:media/virtual-machines-shared-sap-planning-guide/2900-azure-ha-sap-ha.png
[planning-guide-figure-2901]:media/virtual-machines-shared-sap-planning-guide/2901-azure-ha-sap-ha-md.png
[planning-guide-figure-300]:media/virtual-machines-shared-sap-planning-guide/300-vpn-s2s.png
[planning-guide-figure-3000]:media/virtual-machines-shared-sap-planning-guide/3000-sap-ha-on-azure.png
[planning-guide-figure-3200]:media/virtual-machines-shared-sap-planning-guide/3200-sap-ha-with-sql.png
[planning-guide-figure-3201]:media/virtual-machines-shared-sap-planning-guide/3201-sap-ha-with-sql-md.png
[planning-guide-figure-400]:media/virtual-machines-shared-sap-planning-guide/400-vm-services.png
[planning-guide-figure-600]:media/virtual-machines-shared-sap-planning-guide/600-s2s-details.png
[planning-guide-figure-700]:media/virtual-machines-shared-sap-planning-guide/700-decision-tree-deploy-to-azure.png
[planning-guide-figure-800]:media/virtual-machines-shared-sap-planning-guide/800-portal-vm-overview.png
[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

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

[virtual-machines-manage-availability]:../../manage-availability.md


## <a name="terminology-definitions"></a>Definiciones terminológicas

**Alta disponibilidad**: se refiere a un conjunto de tecnologías que minimiza las interrupciones de TI al proporcionar una continuidad empresarial de los servicios de TI mediante componentes redundantes, con tolerancia de errores o protegidos mediante conmutación por error dentro del *mismo* centro de datos. En nuestro caso, el centro de datos se encuentra en una región de Azure.

**Recuperación ante desastres**: también hace referencia a la reducción de la interrupción de servicios de TI y su recuperación, pero en *varios* centros de datos que pueden estar a separados por cientos de kilómetros. En nuestro caso, los centros de datos pueden residir en varias regiones de Azure dentro de la misma región geopolítica o en ubicaciones que el usuario haya establecido como cliente.


## <a name="overview-of-high-availability"></a>Información general sobre la alta disponibilidad
La alta disponibilidad SAP en Azure se puede dividir en tres tipos:

* **Alta disponibilidad de la infraestructura de Azure**: 

    Por ejemplo, la alta disponibilidad del proceso (máquinas virtuales), la red o el almacenamiento, y sus beneficios para aumentar la disponibilidad de las aplicaciones de SAP.

* **Utilización del reinicio de VM de la infraestructura de Azure para lograr una *mayor disponibilidad* de las aplicaciones de SAP**: 

    Si decide no usar funcionalidades como los clústeres de conmutación por error de Windows Server (WSFC) o Pacemaker en Linux, se utiliza el reinicio de Azure VM. Con ello se protegen los sistemas SAP del tiempo de inactividad planeado y no planeado de la infraestructura de servidores físicos de Azure y la plataforma subyacente general de Azure.

* **Alta disponibilidad de las aplicaciones de SAP**: 

    Para lograr la alta disponibilidad completa del sistema SAP, es necesario proteger todos los componentes críticos de este. Por ejemplo:
    * Los servidores de aplicaciones de SAP redundantes.
    * Componentes únicos. Un ejemplo podría ser un componente de único punto de error (SPOF), como una instancia de ASCS/SCS de SAP o un sistema de administración de bases de datos (DBMS).

La alta disponibilidad de SAP en Azure presenta algunas diferencias con respecto a la alta disponibilidad de SAP en un entorno local, tanto físico como virtual. En el documento [Alta disponibilidad y continuidad empresarial de SAP NetWeaver en entornos virtuales con VMware e Hyper-V en Microsoft Windows][sap-ha-bc-virtual-env-hyperv-vmware-white-paper] se describen las configuraciones estándar de alta disponibilidad de SAP en entornos virtualizados en Windows.

No hay ninguna configuración de alta disponibilidad de SAP integrada en SAPInst para Linux como la que existe para Windows. Para obtener más información sobre la alta disponibilidad local de SAP para Linux, vea [Información sobre asociados de alta disponibilidad][sap-ha-partner-information].

## <a name="azure-infrastructure-high-availability"></a>Alta disponibilidad de la infraestructura de Azure

### <a name="sla-for-single-instance-virtual-machines"></a>SLA para las máquinas virtuales de instancia única

Actualmente hay un Acuerdo de Nivel de Servicio del 99,9 % para una única máquina virtual con Premium Storage. Para tener una idea de cómo sería la disponibilidad de una sola máquina virtual, puede compilar el producto de los distintos [Acuerdos de Nivel de Servicio de Azure][azure-sla] disponibles.

La base para el cálculo es de 30 días al mes o 43 200 minutos. Por ejemplo, un tiempo de inactividad del 0,05 % corresponde a 21,6 minutos. Como es habitual, la disponibilidad de los distintos servicios se calcula de la siguiente manera:

(Servicio de disponibilidad n.º 1/100) * (servicio de disponibilidad n.º 2/100) * (servicio de disponibilidad n.º 3/100) \*…

Por ejemplo:

(99,95/100) * (99,9/100) * (99,9/100) = 0,9975 o una disponibilidad general del 99,75 %.

### <a name="multiple-instances-of-virtual-machines-in-the-same-availability-set"></a>Varias instancias de máquinas virtuales en el mismo conjunto de disponibilidad
Para todas las máquinas virtuales que tengan dos o más instancias implementadas en el mismo *conjunto de disponibilidad*, garantizamos conectividad de las máquinas virtuales a una instancia como mínimo al menos durante el 99,95 % del tiempo.

Cuando dos o más máquinas virtuales forman parte del mismo conjunto de disponibilidad, la plataforma Azure subyacente le asigna a cada una un *dominio de actualización* y un *dominio de error*.

* Los **dominios de actualización** garantizan que diferentes máquinas virtuales no se reinician al mismo tiempo durante el mantenimiento planificado de una infraestructura de Azure. Las máquinas virtuales se reinician de una en una.

* Los **dominios de error** garantizan que las máquinas virtuales se implementan en componentes de hardware diferentes que no comparten interruptor de red ni fuente de alimentación. Cuando los servidores, el interruptor de red o la fuente de alimentación experimentan un tiempo de inactividad imprevisto, solo una máquina virtual se verá afectada.

Para obtener más información, vea [Administración de la disponibilidad de las máquinas virtuales Windows en Azure][azure-virtual-machines-manage-availability].

El conjunto de disponibilidad se utiliza para lograr alta disponibilidad de:

* Los servidores de aplicaciones de SAP redundantes.  
* Clústeres con dos o más nodos (máquinas virtuales, por ejemplo) que protegen puntos de error únicos como una instancia de ASCS/SCS de SAP o un administrador de base de datos.


### <a name="azure-availability-zones"></a>Azure Availability Zones
Azure lanzará próximamente [Azure Availability Zones](../../../availability-zones/az-overview.md) en diferentes [regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions/). En las regiones de Azure donde se va a ofrecer zonas de disponibilidad, hay muchos centros de datos en los que el suministro de la fuente de energía, la refrigeración y la red es independiente. El motivo de ofrecer distintas zonas dentro de una única región de Azure es permitirle implementar aplicaciones en dos o tres de estas zonas. Suponiendo que los problemas en las fuentes de energía o la red afectaran únicamente a la infraestructura de una zona de disponibilidad, la implementación de su aplicación dentro de una región de Azure seguiría siendo completamente funcional. En última instancia, con cierta capacidad reducida, dado que algunas máquinas virtuales de una zona podrían perderse. Pero las máquinas virtuales de las otras dos zonas seguirían funcionando. Las regiones de Azure que ofrecen zonas aparecen en [Azure Availability Zones](../../../availability-zones/az-overview.md).

Hay algunas cosas a tener en cuenta a la hora de usar Availability Zones. Cosas como:

- No se pueden implementar conjuntos de disponibilidad de Azure en una zona de disponibilidad. Debe elegir una zona de disponibilidad o un conjunto de disponibilidad como marco de implementación para una máquina virtual.
- No puede usar [Basic Load Balancer](../../../load-balancer/load-balancer-overview.md) para crear soluciones de clúster de conmutación por error basadas en los servicios de clúster de conmutación por error de Windows o en Linux Pacemaker. En su lugar, deberá usar el [SKU Azure Standard Load Balancer](../../../load-balancer/load-balancer-standard-availability-zones.md)
- Azure Availability Zones no proporciona ninguna garantía de una determinada distancia entre las diferentes zonas de una región.
- La latencia de red entre diferentes instancias de Azure Availability Zones en distintas regiones de Azure puede variar de una región a otra. Habrá casos en los que puede, como cliente, ejecutar de manera razonable el nivel de aplicación de SAP implementado en diferentes zonas ya que la latencia de red de una zona a la máquina virtual de DBMS activa es aceptable desde el punto de vista del impacto del proceso empresarial. Sin embargo, también habrá escenarios de cliente en los que la latencia entre la máquina virtual del DBMS de una zona y una instancia de la aplicación de SAP en una máquina virtual de otra zona sea demasiado intrusiva o no aceptable para los procesos empresariales de SAP. Como resultado, las arquitecturas de implementación deben ser diferentes con una arquitectura en modo activo/activo para la aplicación o activo/pasivo si la latencia es demasiado alta.
- El uso de [discos administrados de Azure](https://azure.microsoft.com/services/managed-disks/) es obligatorio para la implementación en Azure Availability Zones 


### <a name="planned-and-unplanned-maintenance-of-virtual-machines"></a>Mantenimiento planeado y no planeado de las máquinas virtuales

Dos tipos de eventos de la plataforma Azure pueden afectar a la disponibilidad de las máquinas virtuales:

* Los eventos de **mantenimiento planeado** son las actualizaciones periódicas realizadas por Microsoft a la plataforma Azure subyacente. Las actualizaciones mejoran la confiabilidad general, el rendimiento y seguridad de la infraestructura de plataforma en la que se ejecutan las máquinas virtuales.

* Los eventos de **mantenimiento no planeado** se producen cuando el hardware o la infraestructura física subyacente a la máquina virtual presentan algún tipo de error. Entre estos podemos encontrar errores de la red local, errores de los discos locales u otros errores a nivel de bastidor. Cuando se detecta un error de este tipo, la plataforma Azure migra automáticamente la máquina virtual del servidor físico en mal estado que la hospeda a un servidor físico con un estado correcto. Estos eventos no son habituales, pero pueden hacer que su máquina virtual se reinicie.

Para obtener más información, vea [Administración de la disponibilidad de las máquinas virtuales Windows en Azure][azure-virtual-machines-manage-availability].

### <a name="azure-storage-redundancy"></a>Redundancia de Azure Storage
Los datos de la cuenta de almacenamiento se replican siempre para garantizar su durabilidad y alta disponibilidad, en cumplimiento del Acuerdo de Nivel de Servicio de Azure Storage, incluso en caso de errores transitorios del hardware.

Dado que Azure Storage mantiene tres imágenes de los datos de forma predeterminada, no es necesario el uso de RAID 5 o RAID 1 en varios discos de Azure.

Para más información, consulte [Replicación de Azure Storage][azure-storage-redundancy].

### <a name="azure-managed-disks"></a>Azure Managed Disks
Los discos Managed Disks son un tipo de recurso de Azure Resource Manager que es recomendable usar en lugar de discos duros virtuales almacenados en las cuentas de Azure Storage. Los discos Managed Disks se alinearán automáticamente con un conjunto de disponibilidad de Azure de la máquina virtual a la que están conectados. Aumentan la disponibilidad de la máquina virtual y de los servicios que se ejecutan en ella.

Para obtener más información, vea [Introducción a los discos administrados de Azure][azure-storage-managed-disks-overview].

Se recomienda usar discos Managed Disks, porque simplifican la implementación y administración de las máquinas virtuales.



## <a name="utilizing-azure-infrastructure-high-availability-to-achieve-higher-availability-of-sap-applications"></a>Uso de la alta disponibilidad de la infraestructura de Azure para lograr la *alta disponibilidad* de las aplicaciones de SAP

Si decide no usar funcionalidades como WSFC o Pacemaker en Linux (actualmente admitido solo para SUSE Linux Enterprise Server [SLES] 12 y versiones posteriores), se utiliza el reinicio de máquina virtual de Azure. Con ello se protegen los sistemas SAP del tiempo de inactividad planeado y no planeado de la infraestructura de servidores físicos de Azure y la plataforma subyacente general de Azure.

Para obtener más información sobre este enfoque, vea [Utilización del reinicio de VM de la infraestructura de Azure para lograr una "mayor disponibilidad" de un sistema de SAP][sap-higher-availability].

## <a name="high-availability-of-sap-applications-on-azure-iaas"></a><a name="baed0eb3-c662-4405-b114-24c10a62954e"></a>Alta disponibilidad de las aplicaciones de SAP en IaaS de Azure

Para lograr la alta disponibilidad completa del sistema SAP, es necesario proteger todos los componentes críticos de este. Por ejemplo:
  * Los servidores de aplicaciones de SAP redundantes.
  * Componentes únicos. Un ejemplo podría ser un componente de único punto de error (SPOF), como una instancia de ASCS/SCS de SAP o un sistema de administración de bases de datos (DBMS).

En la siguiente sección se tratara el tema de cómo lograr alta disponibilidad para los tres componentes críticos del sistema SAP.

### <a name="high-availability-architecture-for-sap-application-servers"></a>Arquitectura de alta disponibilidad en los servidores de aplicaciones de SAP

> Esta sección es aplicable a:
>
> ![Windows][Logo_Windows] Windows como para ![Linux][Logo_Linux] Linux
>

Por lo general, no se necesita una solución de alta disponibilidad específica para las instancias de diálogo y servidores de aplicaciones de SAP. La alta disponibilidad se obtiene mediante redundancia y se configuran varias instancias de diálogo en distintas instancias de Azure Virtual Machines. Debe tener, como mínimo, dos instancias de aplicaciones de SAP instaladas en dos instancias de Azure Virtual Machines.

![Figura 1: Alta disponibilidad en los servidores de aplicaciones de SAP][sap-ha-guide-figure-2000]

_**Ilustración 1:** Alta disponibilidad en los servidores de aplicaciones de SAP_

Es necesario que coloque todas las máquinas virtuales que hospedan instancias de servidores de aplicaciones de SAP en el mismo conjunto de disponibilidad de Azure. Un conjunto de disponibilidad de Azure garantiza lo siguiente:

* No todas las máquinas virtuales forman parte del mismo dominio de actualización.  
    Un dominio de actualización garantiza que las máquinas virtuales no se actualicen al mismo tiempo durante un tiempo de inactividad por mantenimiento planeado.

    La funcionalidad básica, que se basa en diversos dominios de actualización y de error dentro de una unidad de escalado de Azure, ya se ha presentado en la sección sobre [dominios de actualización][planning-guide-3.2.2].

* No todas las máquinas virtuales forman parte del mismo dominio de error.  
    Un dominio de error garantiza que las máquinas virtuales estén implementadas de tal manera que ningún único punto de error afecte la disponibilidad de todas las máquinas virtuales.

El número de dominios de error y de actualización que puede utilizar un conjunto de disponibilidad de Azure dentro de una unidad de escalado de Azure no es infinito. Si agrega continuamente máquinas virtuales a un único conjunto de disponibilidad, llegará un momento en el que dos o más máquinas virtuales coincidan en el mismo dominio de error o actualización.

Si se implementan unas cuantas instancias del servidor de aplicaciones de SAP en sus máquinas virtuales dedicadas, y se parte de la base de que hay cinco dominios de actualización, al final se obtendrá la imagen siguiente. El número máximo real de dominios de error y de actualización dentro de un conjunto de disponibilidad podría cambiar en el futuro:

![Figura 2: Alta disponibilidad de servidores de aplicaciones de SAP en un conjunto de disponibilidad de Azure][planning-guide-figure-3000]
 _**Figura 2:** Alta disponibilidad de servidores de aplicaciones de SAP en un conjunto de disponibilidad de Azure_

Para obtener más información, vea [Administración de la disponibilidad de las máquinas virtuales Windows en Azure][azure-virtual-machines-manage-availability].

Para obtener más información, vea la sección [Conjuntos de disponibilidad de Azure][planning-guide-3.2.3] del documento sobre implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver.

**Solo discos no administrados:** Como la cuenta de almacenamiento de Azure es un punto único de error potencial, es importante tener, como mínimo, 2 cuentas de almacenamiento de Azure, en los cuales hay distribuidas, al menos, 2 máquinas virtuales. En una configuración ideal, los discos de cada máquina virtual que ejecuta una instancia de diálogo de SAP estarían implementados en una cuenta de almacenamiento distinta.

> [!IMPORTANT]
> Es muy recomendable que utilice Azure Managed Disks para las instalaciones de alta disponibilidad de SAP. Los discos Managed Disks se alinean automáticamente con el conjunto de disponibilidad de la máquina virtual a la que están conectados y, por tanto, aumentan la disponibilidad de la máquina virtual y los servicios que se ejecutan en ella.  
>

### <a name="high-availability-architecture-for-an-sap-ascsscs-instance-on-windows"></a>Arquitectura de alta disponibilidad para una instancia de ASCS/SCS de SAP en Windows

> ![Windows][Logo_Windows] Windows
>

Puede usar una solución WSFC para proteger la instancia de ASCS/SCS de SAP. La solución tiene dos variantes:

* **Agrupación en clústeres de la instancia ASCS/SCS de SAP mediante discos compartidos en el clúster**: Para obtener más información sobre esta arquitectura, vea [Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un disco compartido de clúster][sap-high-availability-guide-wsfc-shared-disk].   

* **Agrupación en clústeres de la instancia ASCS/SCS de SAP mediante un recurso compartido de archivos**: Para obtener más información sobre esta arquitectura, vea [Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un recurso compartido de archivos][sap-high-availability-guide-wsfc-file-share].

* **Agrupación en clústeres de la instancia ASCS/SCS de SAP mediante un recurso compartido de SMB de ANF**: Para más información sobre esta arquitectura, consulte [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en Windows con Azure NetApp Files (SMB) para aplicaciones SAP](./high-availability-guide-windows-netapp-files-smb.md).

### <a name="high-availability-architecture-for-an-sap-ascsscs-instance-on-linux"></a>Arquitectura de alta disponibilidad para una instancia de ASCS/SCS de SAP en Linux

> ![Linux][Logo_Linux] Linux
> 
> Para más información sobre la agrupación en clústeres de la instancia de ASCS/SCS de SAP mediante el marco de clústeres de SLES, consulte [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications][sap-suse-ascs-ha]. Para información sobre la arquitectura de alta disponibilidad alternativa en SLES, que no requiere NFS de alta disponibilidad, consulte [Guía de alta disponibilidad para SAP NetWeaver en SUSE Linux Enterprise Server con Azure NetApp Files para aplicaciones SAP][sap-suse-ascs-ha-anf].

Para más información acerca de la agrupación en clústeres de la instancia de ASCS/SCS de SAP mediante la plataforma de Red Hat, consulte [Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver en Red Hat Enterprise Linux](./high-availability-guide-rhel.md).


### <a name="sap-netweaver-multi-sid-configuration-for-a-clustered-sap-ascsscs-instance"></a>Configuración de varios identificadores de seguridad de SAP NetWeaver para una instancia de ASCS/SCS de SAP agrupada en clúster

> ![Windows][Logo_Windows] Windows
> 
> Se admiten varios identificadores de seguridad con WSFC mediante los recursos compartidos de archivos y discos compartidos.
> 
> Para más información sobre la arquitectura de alta disponibilidad con varios identificadores de seguridad en Windows, consulte:

* [Alta disponibilidad con varios identificadores de seguridad de instancia SAP ASCS/SCS para los clústeres de conmutación por error de Windows Server y los recursos compartidos de archivos][sap-ascs-ha-multi-sid-wsfc-file-share]

* [Alta disponibilidad con varios identificadores de seguridad de instancia SAP ASCS/SCS para los clústeres de conmutación por error de Windows Server y el disco compartido][sap-ascs-ha-multi-sid-wsfc-shared-disk]

> ![Linux][Logo_Linux] Linux
> 
> La agrupación en clústeres de varios SID es compatible con los clústeres de Linux Pacemaker para ASCS/ERS de SAP, limitado a **cinco** SID de SAP en el mismo clúster.
> Para más información sobre la arquitectura de alta disponibilidad con varios identificadores de seguridad en Linux, consulte:

* [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications: guía de varios SID](./high-availability-guide-suse-multi-sid.md)
* [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en Red Hat Enterprise Linux para SAP Applications: guía de varios SID](./high-availability-guide-rhel-multi-sid.md)

### <a name="high-availability-dbms-instance"></a>Alta disponibilidad para la instancia de DBMS

DBMS también es un único punto de contacto de un sistema SAP. Debe protegerlo con una solución de alta disponibilidad. En la siguiente ilustración se muestra una solución de alta disponibilidad de SQL Server AlwaysOn en Azure, con clústeres de conmutación por error de Windows Server y el equilibrador de carga interno de Azure. SQL Server AlwaysOn replica los archivos de datos y de registro del sistema de administración de bases de datos (DBMS) con su propia replicación DBMS. En este caso, no se necesitan discos compartidos en el clúster, lo que simplifica toda la configuración.

![Ilustración 3: Ejemplo de DBMS con SAP de alta disponibilidad con SQL Server AlwaysOn][sap-ha-guide-figure-2003]

_**Ilustración 3:** Ejemplo de DBMS con SAP de alta disponibilidad con SQL Server AlwaysOn_

Para más información sobre cómo agrupar clústeres de DBMS de SQL Server en Azure utilizando el modelo de implementación de Azure Resource Manager, consulte estos artículos:

* [Configuración de un grupo de disponibilidad AlwaysOn en Azure Virtual Machines de forma manual mediante Resource Manager][virtual-machines-windows-portal-sql-alwayson-availability-groups-manual]

* [Configuración de un equilibrador de carga interno para un grupo de disponibilidad AlwaysOn en Azure][virtual-machines-windows-portal-sql-alwayson-int-listener]

Para obtener más información sobre cómo agrupar clústeres de DBMS de SAP HANA en Azure con el modelo de implementación de Azure Resource Manager, vea [Alta disponibilidad de SAP HANA en máquinas virtuales de Azure][sap-hana-ha].
