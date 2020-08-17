---
title: 'SAP en Azure: Guía de planeamiento e implementación'
description: Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: MSSedusch
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: d7c59cc1-b2d0-4d90-9126-628f9c7a5538
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 06/23/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 5cd335d34a67cc5a102bde11366813c53770266e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87036342"
---
# <a name="azure-virtual-machines-planning-and-implementation-for-sap-netweaver"></a>Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver

[767598]:https://launchpad.support.sap.com/#/notes/767598
[773830]:https://launchpad.support.sap.com/#/notes/773830
[826037]:https://launchpad.support.sap.com/#/notes/826037
[965908]:https://launchpad.support.sap.com/#/notes/965908
[1031096]:https://launchpad.support.sap.com/#/notes/1031096
[1139904]:https://launchpad.support.sap.com/#/notes/1139904
[1173395]:https://launchpad.support.sap.com/#/notes/1173395
[1245200]:https://launchpad.support.sap.com/#/notes/1245200
[1409604]:https://launchpad.support.sap.com/#/notes/1409604
[1558958]:https://launchpad.support.sap.com/#/notes/1558958
[1585981]:https://launchpad.support.sap.com/#/notes/1585981
[1588316]:https://launchpad.support.sap.com/#/notes/1588316
[1590719]:https://launchpad.support.sap.com/#/notes/1590719
[1597355]:https://launchpad.support.sap.com/#/notes/1597355
[1605680]:https://launchpad.support.sap.com/#/notes/1605680
[1619720]:https://launchpad.support.sap.com/#/notes/1619720
[1619726]:https://launchpad.support.sap.com/#/notes/1619726
[1619967]:https://launchpad.support.sap.com/#/notes/1619967
[1750510]:https://launchpad.support.sap.com/#/notes/1750510
[1752266]:https://launchpad.support.sap.com/#/notes/1752266
[1757924]:https://launchpad.support.sap.com/#/notes/1757924
[1757928]:https://launchpad.support.sap.com/#/notes/1757928
[1758182]:https://launchpad.support.sap.com/#/notes/1758182
[1758496]:https://launchpad.support.sap.com/#/notes/1758496
[1772688]:https://launchpad.support.sap.com/#/notes/1772688
[1814258]:https://launchpad.support.sap.com/#/notes/1814258
[1882376]:https://launchpad.support.sap.com/#/notes/1882376
[1909114]:https://launchpad.support.sap.com/#/notes/1909114
[1922555]:https://launchpad.support.sap.com/#/notes/1922555
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1941500]:https://launchpad.support.sap.com/#/notes/1941500
[1956005]:https://launchpad.support.sap.com/#/notes/1956005
[1973241]:https://launchpad.support.sap.com/#/notes/1973241
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2039619]:https://launchpad.support.sap.com/#/notes/2039619
[2069760]:https://launchpad.support.sap.com/#/notes/2069760
[2121797]:https://launchpad.support.sap.com/#/notes/2121797
[2134316]:https://launchpad.support.sap.com/#/notes/2134316
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2233094]:https://launchpad.support.sap.com/#/notes/2233094
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[azure-cli]:../../../cli-install-nodejs.md
[azure-portal]:https://portal.azure.com
[azure-ps]:/powershell/azure/
[azure-quickstart-templates-github]:https://github.com/Azure/azure-quickstart-templates
[azure-script-ps]:https://go.microsoft.com/fwlink/p/?LinkID=395017
[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md#subscription-limits

[dbms-guide]:dbms-guide.md
[dbms-guide-2.1]:dbms-guide.md#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f
[dbms-guide-2.2]:dbms-guide.md#c8e566f9-21b7-4457-9f7f-126036971a91
[dbms-guide-2.3]:dbms-guide.md#10b041ef-c177-498a-93ed-44b3441ab152
[dbms-guide-2]:dbms-guide.md#65fa79d6-a85f-47ee-890b-22e794f51a64
[dbms-guide-3]:dbms-guide.md#871dfc27-e509-4222-9370-ab1de77021c3
[dbms-guide-5.5.1]:dbms-guide.md#0fef0e79-d3fe-4ae2-85af-73666a6f7268
[dbms-guide-5.5.2]:dbms-guide.md#f9071eff-9d72-4f47-9da4-1852d782087b
[dbms-guide-5.6]:dbms-guide.md#1b353e38-21b3-4310-aeb6-a77e7c8e81c8
[dbms-guide-5.8]:dbms-guide.md#9053f720-6f3b-4483-904d-15dc54141e30
[dbms-guide-5]:dbms-guide.md#3264829e-075e-4d25-966e-a49dad878737
[dbms-guide-8.4.1]:dbms-guide.md#b48cfe3b-48e9-4f5b-a783-1d29155bd573
[dbms-guide-8.4.2]:dbms-guide.md#23c78d3b-ca5a-4e72-8a24-645d141a3f5d
[dbms-guide-8.4.3]:dbms-guide.md#77cd2fbb-307e-4cbf-a65f-745553f72d2c
[dbms-guide-8.4.4]:dbms-guide.md#f77c1436-9ad8-44fb-a331-8671342de818
[dbms-guide-900-sap-cache-server-on-premises]:dbms-guide.md#642f746c-e4d4-489d-bf63-73e80177a0a8

[dbms-guide-figure-100]:media/virtual-machines-shared-sap-dbms-guide/100_storage_account_types.png
[dbms-guide-figure-200]:media/virtual-machines-shared-sap-dbms-guide/200-ha-set-for-dbms-ha.png
[dbms-guide-figure-300]:media/virtual-machines-shared-sap-dbms-guide/300-reference-config-iaas.png
[dbms-guide-figure-400]:media/virtual-machines-shared-sap-dbms-guide/400-sql-2012-backup-to-blob-storage.png
[dbms-guide-figure-500]:media/virtual-machines-shared-sap-dbms-guide/500-sql-2012-backup-to-blob-storage-different-containers.png
[dbms-guide-figure-600]:media/virtual-machines-shared-sap-dbms-guide/600-iaas-maxdb.png
[dbms-guide-figure-700]:media/virtual-machines-shared-sap-dbms-guide/700-livecach-prod.png
[dbms-guide-figure-800]:media/virtual-machines-shared-sap-dbms-guide/800-azure-vm-sap-content-server.png
[dbms-guide-figure-900]:media/virtual-machines-shared-sap-dbms-guide/900-sap-cache-server-on-premises.png

[deployment-guide]:deployment-guide.md
[deployment-guide-2.2]:deployment-guide.md#42ee2bdb-1efc-4ec7-ab31-fe4c22769b94
[deployment-guide-3.1.2]:deployment-guide.md#3688666f-281f-425b-a312-a77e7db2dfab
[deployment-guide-3.2]:deployment-guide.md#db477013-9060-4602-9ad4-b0316f8bb281
[deployment-guide-3.3]:deployment-guide.md#54a1fc6d-24fd-4feb-9c57-ac588a55dff2
[deployment-guide-3.4]:deployment-guide.md#a9a60133-a763-4de8-8986-ac0fa33aa8c1
[deployment-guide-3]:deployment-guide.md#b3253ee3-d63b-4d74-a49b-185e76c4088e
[deployment-guide-4.1]:deployment-guide.md#604bcec2-8b6e-48d2-a944-61b0f5dee2f7
[deployment-guide-4.2]:deployment-guide.md#7ccf6c3e-97ae-4a7a-9c75-e82c37beb18e
[deployment-guide-4.3]:deployment-guide.md#31d9ecd6-b136-4c73-b61e-da4a29bbc9cc
[deployment-guide-4.4.2]:deployment-guide.md#6889ff12-eaaf-4f3c-97e1-7c9edc7f7542
[deployment-guide-4.4]:deployment-guide.md#c7cbb0dc-52a4-49db-8e03-83e7edc2927d
[deployment-guide-4.5.1]:deployment-guide.md#987cf279-d713-4b4c-8143-6b11589bb9d4
[deployment-guide-4.5.2]:deployment-guide.md#408f3779-f422-4413-82f8-c57a23b4fc2f
[deployment-guide-4.5]:deployment-guide.md#d98edcd3-f2a1-49f7-b26a-07448ceb60ca
[deployment-guide-5.1]:deployment-guide.md#bb61ce92-8c5c-461f-8c53-39f5e5ed91f2
[deployment-guide-5.2]:deployment-guide.md#e2d592ff-b4ea-4a53-a91a-e5521edb6cd1
[deployment-guide-5.3]:deployment-guide.md#fe25a7da-4e4e-4388-8907-8abc2d33cfd8

[deployment-guide-configure-monitoring-scenario-1]:deployment-guide.md#ec323ac3-1de9-4c3a-b770-4ff701def65b
[deployment-guide-configure-proxy]:deployment-guide.md#baccae00-6f79-4307-ade4-40292ce4e02d
[deployment-guide-figure-100]:media/virtual-machines-shared-sap-deployment-guide/100-deploy-vm-image.png
[deployment-guide-figure-1000]:media/virtual-machines-shared-sap-deployment-guide/1000-service-properties.png
[deployment-guide-figure-11]:deployment-guide.md#figure-11
[deployment-guide-figure-1100]:media/virtual-machines-shared-sap-deployment-guide/1100-azperflib.png
[deployment-guide-figure-1200]:media/virtual-machines-shared-sap-deployment-guide/1200-cmd-test-login.png
[deployment-guide-figure-1300]:media/virtual-machines-shared-sap-deployment-guide/1300-cmd-test-executed.png
[deployment-guide-figure-14]:deployment-guide.md#figure-14
[deployment-guide-figure-1400]:media/virtual-machines-shared-sap-deployment-guide/1400-azperflib-error-servicenotstarted.png
[deployment-guide-figure-300]:media/virtual-machines-shared-sap-deployment-guide/300-deploy-private-image.png
[deployment-guide-figure-400]:media/virtual-machines-shared-sap-deployment-guide/400-deploy-using-disk.png
[deployment-guide-figure-5]:deployment-guide.md#figure-5
[deployment-guide-figure-50]:media/virtual-machines-shared-sap-deployment-guide/50-forced-tunneling-suse.png
[deployment-guide-figure-500]:media/virtual-machines-shared-sap-deployment-guide/500-install-powershell.png
[deployment-guide-figure-6]:deployment-guide.md#figure-6
[deployment-guide-figure-600]:media/virtual-machines-shared-sap-deployment-guide/600-powershell-version.png
[deployment-guide-figure-7]:deployment-guide.md#figure-7
[deployment-guide-figure-700]:media/virtual-machines-shared-sap-deployment-guide/700-install-powershell-installed.png
[deployment-guide-figure-760]:media/virtual-machines-shared-sap-deployment-guide/760-azure-cli-version.png
[deployment-guide-figure-900]:media/virtual-machines-shared-sap-deployment-guide/900-cmd-update-executed.png
[deployment-guide-figure-azure-cli-installed]:deployment-guide.md#402488e5-f9bb-4b29-8063-1c5f52a892d0
[deployment-guide-figure-azure-cli-version]:deployment-guide.md#0ad010e6-f9b5-4c21-9c09-bb2e5efb3fda
[deployment-guide-install-vm-agent-windows]:deployment-guide.md#b2db5c9a-a076-42c6-9835-16945868e866
[deployment-guide-troubleshooting-chapter]:deployment-guide.md#564adb4f-5c95-4041-9616-6635e83a810b

[deploy-template-cli]:../../../resource-group-template-deploy-cli.md
[deploy-template-portal]:../../../resource-group-template-deploy-portal.md
[deploy-template-powershell]:../../../resource-group-template-deploy.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md
[getting-started-dbms]:get-started.md#1343ffe1-8021-4ce6-a08d-3a1553a4db82
[getting-started-deployment]:get-started.md#6aadadd2-76b5-46d8-8713-e8d63630e955
[getting-started-planning]:get-started.md#3da0389e-708b-4e82-b2a2-e92f132df89c

[getting-started-windows-classic]:../../virtual-machines-windows-classic-sap-get-started.md
[getting-started-windows-classic-dbms]:../../virtual-machines-windows-classic-sap-get-started.md#c5b77a14-f6b4-44e9-acab-4d28ff72a930
[getting-started-windows-classic-deployment]:../../virtual-machines-windows-classic-sap-get-started.md#f84ea6ce-bbb4-41f7-9965-34d31b0098ea
[getting-started-windows-classic-dr]:../../virtual-machines-windows-classic-sap-get-started.md#cff10b4a-01a5-4dc3-94b6-afb8e55757d3
[getting-started-windows-classic-ha-sios]:../../virtual-machines-windows-classic-sap-get-started.md#4bb7512c-0fa0-4227-9853-4004281b1037
[getting-started-windows-classic-planning]:../../virtual-machines-windows-classic-sap-get-started.md#f2a5e9d8-49e4-419e-9900-af783173481c

[ha-guide-classic]:https://go.microsoft.com/fwlink/?LinkId=613056

[install-extension-cli]:virtual-machines-linux-enable-aem.md

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png

[msdn-set-Azvmaemextension]:https://msdn.microsoft.com/library/azure/mt670598.aspx

[planning-guide]:planning-guide.md  
[planning-guide-1.2]:planning-guide.md#e55d1e22-c2c8-460b-9897-64622a34fdff
[planning-guide-11]:planning-guide.md#7cf991a1-badd-40a9-944e-7baae842a058
[planning-guide-11.4.1]:planning-guide.md#5d9d36f9-9058-435d-8367-5ad05f00de77
[planning-guide-11.5]:planning-guide.md#4e165b58-74ca-474f-a7f4-5e695a93204f
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

[powershell-install-configure]:https://docs.microsoft.com/powershell/azure/install-az-ps
[resource-group-authoring-templates]:../../../resource-group-authoring-templates.md
[resource-group-overview]:../../../azure-resource-manager/management/overview.md
[resource-groups-networking]:../../../networking/networking-overview.md
[sap-pam]:https://support.sap.com/pam
[sap-templates-2-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-2-tier-os-disk]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-disk%2Fazuredeploy.json
[sap-templates-2-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-image%2Fazuredeploy.json
[sap-templates-3-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-3-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-user-image%2Fazuredeploy.json
[storage-azure-cli]:../../../storage/common/storage-azure-cli.md
[storage-azure-cli-copy-blobs]:../../../storage/common/storage-azure-cli.md#copy-blobs
[storage-introduction]:../../../storage/common/storage-introduction.md
[storage-powershell-guide-full-copy-vhd]:../../../storage/common/storage-powershell-guide-full.md
[storage-premium-storage-preview-portal]:../../windows/disks-types.md
[storage-redundancy]:../../../storage/common/storage-redundancy.md
[storage-scalability-targets]:../../../storage/common/scalability-targets-standard-accounts.md
[storage-use-azcopy]:../../../storage/common/storage-use-azcopy.md
[template-201-vm-from-specialized-vhd]:https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-from-specialized-vhd
[templates-101-simple-windows-vm]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-simple-windows-vm
[templates-101-vm-from-user-image]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image
[virtual-machines-linux-attach-disk-portal]:../../linux/attach-disk-portal.md
[virtual-machines-azure-resource-manager-architecture]:../../../resource-manager-deployment-model.md
[virtual-machines-Az-versus-azuresm]:virtual-machines-linux-compare-deployment-models.md
[virtual-machines-windows-classic-configure-oracle-data-guard]:../../virtual-machines-windows-classic-configure-oracle-data-guard.md
[virtual-machines-linux-cli-deploy-templates]:../../linux/cli-deploy-templates.md
[virtual-machines-deploy-rmtemplates-powershell]:../../virtual-machines-windows-ps-manage.md
[virtual-machines-linux-agent-user-guide]:../../extensions/agent-linux.md
[virtual-machines-linux-agent-user-guide-command-line-options]:../../extensions/agent-windows.md#command-line-options
[virtual-machines-linux-capture-image]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager-capture]:../../linux/capture-image.md#step-2-capture-the-vm
[virtual-machines-windows-capture-image-resource-manager]:../../windows/capture-image.md
[virtual-machines-windows-capture-image]:../../virtual-machines-windows-generalize-vhd.md
[virtual-machines-windows-capture-image-prepare-the-vm-for-image-capture]:../../virtual-machines-windows-generalize-vhd.md
[virtual-machines-linux-configure-raid]:../../linux/configure-raid.md
[virtual-machines-linux-configure-lvm]:../../linux/configure-lvm.md
[virtual-machines-linux-classic-create-upload-vhd-step-1]:../../virtual-machines-linux-classic-create-upload-vhd.md#step-1-prepare-the-image-to-be-uploaded
[virtual-machines-linux-create-upload-vhd-suse]:../../linux/suse-create-upload-vhd.md
[virtual-machines-linux-create-upload-vhd-oracle]:../../linux/oracle-create-upload-vhd.md
[virtual-machines-linux-redhat-create-upload-vhd]:../../linux/redhat-create-upload-vhd.md
[virtual-machines-linux-how-to-attach-disk]:../../linux/add-disk.md
[virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux]:../../linux/add-disk.md#connect-to-the-linux-vm-to-mount-the-new-disk
[virtual-machines-linux-tutorial]:../../linux/quick-create-cli.md
[virtual-machines-linux-update-agent]:../../linux/update-agent.md
[virtual-machines-manage-availability]:../../linux/manage-availability.md
[virtual-machines-ps-create-preconfigure-windows-resource-manager-vms]:virtual-machines-windows-create-powershell.md
[virtual-machines-sizes-linux]:../../linux/sizes.md
[virtual-machines-sizes-windows]:../../windows/sizes.md
[virtual-machines-windows-classic-ps-sql-alwayson-availability-groups]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups.md
[virtual-machines-windows-classic-ps-sql-int-listener]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener.md
[virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions]:./../../windows/sql/virtual-machines-windows-sql-high-availability-dr.md
[virtual-machines-sql-server-infrastructure-services]:./../../windows/sql/virtual-machines-windows-sql-server-iaas-overview.md
[virtual-machines-sql-server-performance-best-practices]:./../../windows/sql/virtual-machines-windows-sql-performance.md
[virtual-machines-upload-image-windows-resource-manager]:../../virtual-machines-windows-upload-image.md
[virtual-machines-windows-tutorial]:../../virtual-machines-windows-hero-tutorial.md
[virtual-machines-workload-template-sql-alwayson]:https://azure.microsoft.com/documentation/templates/sql-server-2014-alwayson-dsc/
[virtual-network-deploy-multinic-arm-cli]:../../linux/multiple-nics.md
[virtual-network-deploy-multinic-arm-ps]:../../windows/multiple-nics.md
[virtual-network-deploy-multinic-arm-template]:../../../virtual-network/template-samples.md
[virtual-networks-configure-vnet-to-vnet-connection]:../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md
[virtual-networks-create-vnet-arm-pportal]:../../../virtual-network/manage-virtual-network.md#create-a-virtual-network
[virtual-networks-manage-dns-in-vnet]:../../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md
[virtual-networks-multiple-nics-windows]:../../windows/multiple-nics.md
[virtual-networks-multiple-nics-linux]:../../linux/multiple-nics.md
[virtual-networks-nsg]:../../../virtual-network/security-overview.md
[virtual-networks-reserved-private-ip]:../../../virtual-network/virtual-networks-static-private-ip-arm-ps.md
[virtual-networks-static-private-ip-arm-pportal]:../../../virtual-network/virtual-networks-static-private-ip-arm-pportal.md
[virtual-networks-udr-overview]:../../../virtual-network/virtual-networks-udr-overview.md
[vpn-gateway-about-vpn-devices]:../../../vpn-gateway/vpn-gateway-about-vpn-devices.md
[vpn-gateway-create-site-to-site-rm-powershell]:../../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md
[vpn-gateway-cross-premises-options]:../../../vpn-gateway/vpn-gateway-about-vpngateways.md
[vpn-gateway-site-to-site-create]:../../../vpn-gateway/vpn-gateway-site-to-site-create.md
[vpn-gateway-vpn-faq]:../../../vpn-gateway/vpn-gateway-vpn-faq.md
[xplat-cli]:../../../cli-install-nodejs.md
[xplat-cli-azure-resource-manager]:../../../xplat-cli-azure-resource-manager.md
[capture-image-linux-step-2-create-vm-image]:../../linux/capture-image.md#step-2-create-vm-image



Microsoft Azure permite a las empresas adquirir recursos de proceso y almacenamiento en un tiempo mínimo sin ciclos de adquisición prolongados. El servicio Azure Virtual Machines permite a las empresas implementar aplicaciones clásicas, como aplicaciones basadas en SAP NetWeaver, en Azure y extender su confiabilidad y disponibilidad sin tener que disponer de más recursos de forma local. Los servicios Azure Virtual Machines también admiten la conectividad entre locales, que permite a las empresas integrar activamente Azure Virtual Machines en sus dominios locales, nubes privadas e infraestructura del sistema SAP.
Estas notas del producto describen los fundamentos de las máquinas virtuales de Microsoft Azure y proporcionan un tutorial sobre los aspectos que tener en cuenta al planear e implementar instalaciones de SAP NetWeaver en Azure y, por lo tanto, es el documento que se debe leer antes de iniciar las implementaciones reales de SAP NetWeaver en Azure.
El documento complementa la Documentación de instalación de SAP y las Notas de SAP, que representan los principales recursos para las instalaciones e implementaciones de software de SAP en las plataformas proporcionadas.

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

## <a name="summary"></a>Resumen
La informática en la nube es un término ampliamente usado que está cobrando cada vez más importancia en la industria de TI, desde pequeñas empresas hasta las grandes corporaciones y multinacionales.

Microsoft Azure es la plataforma de Cloud Services de Microsoft que ofrece una amplia gama de nuevas posibilidades. Ahora los clientes pueden aprovisionar y desaprovisionar rápidamente aplicaciones como un servicio en la nube, por lo que no se verán limitados por restricciones técnicas o presupuestarias. En lugar de invertir tiempo y presupuesto en infraestructura de hardware, las empresas pueden centrarse en la aplicación, en los procesos empresariales y en sus beneficios para clientes y usuarios.

Con los servicios de máquinas virtuales de Microsoft Azure, Microsoft ofrece una completa plataforma de infraestructura como servicio (IaaS). Las aplicaciones de basadas en SAP NetWeaver son compatibles en Azure Virtual Machines (IaaS). En estas notas del producto se describe cómo planear e implementar aplicaciones basadas en SAP NetWeaver en Microsoft Azure como plataforma preferida.

El documento en sí se centrará en dos aspectos principales:

* La primera parte describe dos modelos de implementación compatibles para las aplicaciones basadas en SAP NetWeaver en Azure. También describe el control general de Azure desde el punto de vista de las implementaciones de SAP.
* La segunda parte detalla la implementación de los diferentes escenarios descritos en la primera.

Par recursos adicionales, consulte el capítulo [Recursos][planning-guide-1.2] de este documento.

### <a name="definitions-upfront"></a>Lista de definiciones
En el documento se utilizarán los términos siguientes:

* IaaS: Infraestructura como servicio
* PaaS: Plataforma como servicio
* SaaS: Software como servicio
* Componente de SAP: una aplicación de SAP individual, como ECC, BW, Solution Manager o S/4HANA.  Los componentes de SAP pueden basarse en tecnologías tradicionales, como ABAP o Java, o en una aplicación no basada en NetWeaver, como Business Objects.
* Entorno de SAP: uno o varios componentes de SAP agrupados lógicamente para desempeñar una función empresarial, como desarrollo, control de calidad, aprendizaje, recuperación ante desastres o producción.
* Infraestructura de SAP: Este término hace referencia a todos los recursos de SAP de la infraestructura de TI de un cliente. La infraestructura de SAP incluye todos los entornos, tanto los que son de producción como los que no.
* Sistema SAP: La combinación de la capa de DBMS y la capa de aplicación de, por ejemplo, un sistema de desarrollo SAP ERP, un sistema de prueba SAP BW, un sistema de producción SAP CRM, etc. En las implementaciones de Azure no se admite la división de estas dos capas entre la infraestructura local y de Azure. Esto significa que un sistema SAP debe implementarse de forma local o en Azure, pero no en ambos. Sin embargo, los diferentes sistemas de una infraestructura de SAP pueden implementarse en Azure o de forma local. Por ejemplo, pueden implementarse sistemas de pruebas y de desarrollo de SAP CRM en Azure, a la vez que se implementa el sistema de producción de forma local.
* Entre locales o híbrida: describe un escenario donde se implementan máquinas virtuales en una suscripción de Azure con conexión de sitio a sitio, entre varios sitios o de ExpressRoute entre los centros de datos locales y Azure. En la documentación habitual de Azure, este tipo de implementaciones se describen como escenarios entre locales o híbridos. La razón tras la conexión es extender los dominios locales, Active Directory/OpenLDAP locales y DNS local a Azure. La infraestructura local se extiende a los recursos de Azure de la suscripción. Con esta extensión, las máquinas virtuales pueden formar parte del dominio local. Los usuarios del dominio local pueden tener acceso a los servidores y ejecutar servicios en esas máquinas virtuales (por ejemplo, servicios de DBMS). Es posible la comunicación y resolución de nombres entre máquinas virtuales implementadas de forma local y en Azure. Este es el caso más común y casi exclusivo de implementación de recursos de SAP en Azure. Para más información, consulte [este][vpn-gateway-cross-premises-options] artículo y [este][vpn-gateway-site-to-site-create].
* Extensión de Azure Monitor, Supervisión mejorada y Extensión de Azure para SAP: se usan para describir exactamente el mismo elemento. Describe una extensión de máquina virtual que debe implementar el usuario para proporcionar algunos datos básicos sobre la infraestructura de Azure al agente del host de SAP. Puede que las notas de SAP hagan referencia a ella como Extensión de supervisión o Supervisión mejorada. En Azure, haremos referencia a ella como **Extensión de Azure para SAP.**

> [!NOTE]
> En sistemas de producción SAP, se admiten implementaciones entre locales o híbridos de sistemas SAP donde Azure Virtual Machines que ejecuten sistemas SAP pertenezcan a un dominio local. Las configuraciones entre locales o híbridas se admiten para la implementación completa o parcial de infraestructuras de SAP en Azure. Se requiere que esas máquinas virtuales formen parte del dominio y ADS/OpenLDAP locales incluso al ejecutar una infraestructura completa de SAP en Azure. 
>
>



### <a name="resources"></a><a name="e55d1e22-c2c8-460b-9897-64622a34fdff"></a>Recursos
El punto de entrada para la carga de trabajo SAP en la documentación de Azure se encuentra [aquí](./get-started.md). A partir de este punto de entrada, encontrará muchos artículos que tratan los temas siguientes:

- SAP NetWeaver y Business One en Azure
- Guías de DBMS SAP para varios sistemas DBMS en Azure
- Alta disponibilidad y recuperación ante desastres para carga de trabajo de SAP en Azure
- Guía específica para ejecutar SAP HANA en Azure
- Guía específica de instancias grandes de HANA de Azure para DBMS de SAP HANA 


> [!IMPORTANT]
> Siempre que sea posible, se usa un vínculo a las guías de instalación de SAP u otra documentación de SAP a las que se hace referencia (referencia InstGuide-01, consulte <http://service.sap.com/instguides>). En cuanto a los requisitos previos, el proceso de instalación o los detalles de la funcionalidad específica de SAP, las guías y la documentación de SAP deben leerse detenidamente, ya que los documentos de Microsoft solo tratan tareas específicas para software de SAP instalado y en ejecución en una instancia de Microsoft Azure Virtual Machines.
>
>

Las siguientes notas de SAP están relacionadas con el tema de SAP en Azure:

| Número de nota | Título |
| --- | --- |
| [1928533] |SAP Applications on Azure: Supported Products and Sizing (Aplicaciones de SAP en Microsoft Azure con Base de datos de Oracle: versiones y tamaños compatibles) |
| [2015553] |SAP on Microsoft Azure: Support Prerequisites (Requisitos previos de soporte técnico de SAP en Microsoft Azure) |
| [1999351] |Troubleshooting Enhanced Azure Monitoring for SAP (Solución de problemas de la supervisión mejorada de Azure para SAP) |
| [2178632] |Key Monitoring Metrics for SAP on Microsoft Azure (Métricas de supervisión clave para SAP en Microsoft Azure) |
| [1409604] |Virtualization on Windows: Enhanced Monitoring (Virtualización en Windows: supervisión mejorada) |
| [2191498] |SAP on Linux with Azure: Enhanced Monitoring (Virtualización en Windows: supervisión mejorada) |
| [2243692] |Linux on Microsoft Azure (IaaS) VM: SAP license issues (Linux y máquinas virtuales de Microsoft Azure (IaaS): problemas de licencia de SAP) |
| [1984787] |SUSE LINUX Enterprise Server 12: Notas de instalación |
| [2002167] |Red Hat Enterprise Linux 7.x: Instalación y actualización |
| [2069760] |Instalación y actualización de SAP en Oracle Linux 7.x |
| [1597355] |Recomendación de espacio de intercambio para Linux |

Consulte también la [wiki de SCN](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes), que contiene todas las notas de SAP para Linux.

En [este artículo][azure-resource-manager/management/azure-subscription-service-limits-subscription] se exponen las limitaciones generales predeterminadas y el límite máximo de suscripciones de Azure.

## <a name="possible-scenarios"></a>Escenarios posibles
Con frecuencia, SAP se considera una de las aplicaciones críticas en las empresas. La arquitectura y las operaciones de estas aplicaciones suelen resultar complejas y es importante asegurarse de que se cumplen los requisitos de disponibilidad y rendimiento.

Por lo tanto, las empresas tienen que pensar detenidamente qué proveedor de nube van a elegir para ejecutar estos procesos de negocio críticos para la empresa. Azure es la plataforma de nube pública ideal para aplicaciones SAP críticas para la empresa y los procesos empresariales. Dada la amplia variedad de infraestructura de Azure, hoy en día casi todos los sistemas SAP NetWeaver y S/4HANA existentes se pueden hospedar en Azure. Azure proporciona máquinas virtuales con muchos terabytes de memoria y más de 200 CPU. Además de eso, Azure ofrece el servicio de [instancias grandes de HANA](./hana-overview-architecture.md), que permite implementaciones de HANA de escalado vertical de hasta 24 TB e implementaciones de SAP HANA de escalado horizontal de hasta 120 TB. En la actualidad se puede establecer que casi todos los escenarios de SAP locales se pueden ejecutar también en Azure. 

Para una descripción general de los escenarios y algunos escenarios no admitidos, consulte el documento [Carga de trabajo de SAP en escenarios admitidos en máquinas virtuales de Azure](./sap-planning-supported-configurations.md).

Compruebe estos escenarios y algunas de las condiciones con nombre que no se admiten en la documentación a la que se hace referencia a lo largo del planeamiento y el desarrollo de la arquitectura que desea implementar en Azure.

En general, el patrón de implementación más común es un escenario entre locales, como el que se muestra:

![VPN con conectividad de sitio a sitio (entre locales)][planning-guide-figure-300]

El motivo por el que muchos clientes pueden aplicar un patrón de implementación entre locales es que es más transparente para que todas las aplicaciones se extiendan en el entorno local a Azure mediante Azure ExpressRoute y traten Azure como centro de recursos virtual. A medida que se mueven más recursos a Azure, aumentarán la infraestructura de red y la infraestructura implementada de Azure y los recursos locales se reducirán en consecuencia. Todo transparente para los usuarios y las aplicaciones.

Para implementar correctamente sistemas SAP en una IaaS general o en la de Azure, es importante comprender las diferencias significativas entre las ofertas de outsourcers o proveedores de servicios de hosting tradicionales y las ofertas de IaaS. Mientras que el outsourcer o el proveedor de servicios de hosting tradicional adapta la infraestructura (tipo de red, almacenamiento y servidor) a la carga de trabajo que el cliente desea hospedar, en realidad es responsabilidad del cliente o del asociado caracterizar la carga de trabajo y elegir los componentes de Azure adecuados para las máquinas virtuales, el almacenamiento y la red para implementaciones de IaaS.

Con el fin de recopilar datos para el planeamiento de la implementación en Azure, es importante:

- Evaluar qué productos de SAP se admiten en ejecución en máquinas virtuales de Azure.
- Evaluar qué versiones específicas del sistema operativo son compatibles con máquinas virtuales de Azure específicas para esos productos de SAP.
- Evaluar qué versiones de DBMS son compatibles con los productos de SAP con máquinas virtuales de Azure específicas.
- Evaluar si algunas de las versiones necesarias de SO/DBMS requieren que cree la versión de SAP, la actualización del paquete de soporte técnico y las actualizaciones del kernel para obtener una configuración compatible.
- Evaluar si necesita cambiar a sistemas operativos distintos para implementar en Azure.

En el artículo [¿Qué software de SAP es compatible con las implementaciones de Azure?](./sap-supported-product-on-azure.md) se explican los detalles de los componentes de SAP admitidos en Azure, las unidades de infraestructura de Azure compatibles y las versiones de sistema operativo y de DBMS relacionadas. Los resultados obtenidos de la evaluación de las versiones válidas de SAP, del sistema operativo y de las versiones de DBMS tienen un gran impacto en los esfuerzos de mover sistemas SAP a Azure. Los resultados de esta evaluación van a definir si es necesario llevar a cabo esfuerzos de preparación significativos en los casos en los que se necesitan actualizaciones de versiones de SAP o cambios de los sistemas operativos.


## <a name="azure-regions"></a><a name="be80d1b9-a463-4845-bd35-f4cebdb5424a"></a>Regiones de Azure
Los servicios de Azure de Microsoft se recopilan en regiones de Azure. Una región de Azure es una colección de centros de datos que contienen el hardware y la infraestructura que ejecutan y hospedan los distintos servicios de Azure. Esta infraestructura incluye un gran número de nodos que funcionan como nodos de proceso o de almacenamiento, o bien que ejecutan funcionalidades de red. 

Para obtener una lista de las diferentes regiones de Azure, vea el artículo [Zonas geográficas de Azure](https://azure.microsoft.com/global-infrastructure/geographies/). No todas las regiones de Azure ofrecen los mismos servicios. Dependiendo del producto de SAP que quiera ejecutar y del sistema operativo y el DBMS relacionados con este, puede acabar en una situación en la cual una región determinada no ofrece los tipos de máquina virtual que necesita. Esto ocurre particularmente a la hora de ejecutar SAP HANA, donde normalmente se necesitan máquinas virtuales de las series M/MV2. Estas familias de máquinas virtuales se implementan solo en un subconjunto de las regiones. Se puede averiguar qué máquinas virtuales, tipos y tipos de almacenamiento de Azure exactos (u otros servicios de Azure) están disponibles en las regiones con la ayuda del sitio [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/). Cuando empiece el planeamiento y tenga en mente ciertas regiones como candidatas de región principal, así como la región secundaria en última instancia, deberá investigar primero si los servicios necesarios están disponibles en esas regiones. 

### <a name="availability-zones"></a>Zonas de disponibilidad
En varias de las regiones de Azure hay implementado un concepto denominado zonas de disponibilidad. Las zonas de disponibilidad son ubicaciones separadas físicamente dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Por ejemplo, si se implementan dos máquinas virtuales en dos zonas de disponibilidad de Azure y, asimismo, se implementa un marco de alta disponibilidad para el sistema DBMS de SAP o los servicios centrales de SAP, se obtendrá el mejor Acuerdo de Nivel de Servicio en Azure. Para conocer las particularidades de este SLA en Azure, consulte la versión más reciente de los [SLA para máquinas virtuales](https://azure.microsoft.com/support/legal/sla/virtual-machines/). Dado que el uso de regiones de Azure se ha desarrollado y extendido vertiginosamente en los últimos años, la topología de las regiones de Azure, el número de centros de recursos físicos, la distancia entre esos centros de recursos y la distancia entre las zonas de disponibilidad de Azure pueden ser muy variables. Y esto sucede también con la latencia de red.

El principio de zona de disponibilidad no es válido en el servicio específico de HANA de [instancias grandes de HANA](./hana-overview-architecture.md). Encontrará los Acuerdos de Nivel de Servicio del servicio de instancias grandes de HANA en el artículo [SLA para SAP HANA e instancias grandes de Azure](https://azure.microsoft.com/support/legal/sla/sap-hana-large/) 


### <a name="fault-domains"></a><a name="df49dc09-141b-4f34-a4a2-990913b30358"></a>Dominios de error
Los dominios de error representan una unidad física de error. Están estrechamente relacionados con la infraestructura física en los centros de datos y, si bien un bastidor o blade físico puede considerarse un dominio de error, no hay ninguna relación directa entre ambos.

Cuando se implementan varias máquinas virtuales como parte de un sistema SAP en los servicios de Microsoft Azure Virtual Machines, se puede influir en el controlador de tejido de Azure para implementar su aplicación en diferentes dominios de error, lo cual cumple los requisitos de los SLA de disponibilidad. Sin embargo, no se tienen el control de la distribución de los dominios de error en una unidad de escalado de Azure (colección de cientos de nodos de proceso o nodos de almacenamiento y redes) ni de la asignación de máquinas virtuales a un dominio de error específico. Para ordenar al controlador de tejido de Azure que implemente un conjunto de máquinas virtuales en diferentes dominios de error, debe asignar un conjunto de disponibilidad de Azure a las máquinas virtuales en tiempo de implementación. Para más información sobre los conjuntos de disponibilidad de Azure, consulte el capítulo [Conjuntos de disponibilidad de Azure][planning-guide-3.2.3] de este documento.


### <a name="upgrade-domains"></a><a name="fc1ac8b2-e54a-487c-8581-d3cc6625e560"></a>Dominios de actualización
Los dominios de actualización representan una unidad lógica que ayuda a determinar cómo se actualiza una máquina virtual dentro de un sistema SAP, compuesto por instancias de SAP ejecutadas en varias máquinas virtuales. Cuando se produzca una actualización, Microsoft Azure actualiza estos dominios de actualización uno a uno. Al distribuir las máquinas virtuales en tiempo de implementación en diferentes dominios de actualización, se puede proteger parcialmente el sistema SAP frente a posibles tiempos de inactividad. Para forzar que Azure implemente las máquinas virtuales de un sistema SAP distribuidas en diferentes dominios de actualización, se debe establecer un atributo específico durante la implementación de cada máquina virtual. De un modo similar a los dominios de error, una unidad de escalado de Azure se divide en varios dominios de actualización. Para ordenar al controlador de tejido de Azure que implemente un conjunto de máquinas virtuales en diferentes dominios de actualización, debe asignar un conjunto de disponibilidad de Azure a las máquinas virtuales en tiempo de implementación. Para más información sobre los conjuntos de disponibilidad de Azure, consulte el capítulo [Conjuntos de disponibilidad de Azure][planning-guide-3.2.3] que aparece a continuación.


### <a name="azure-availability-sets"></a><a name="18810088-f9be-4c97-958a-27996255c665"></a>Conjuntos de disponibilidad de Azure
El controlador de tejido de Azure distribuye las instancias de Azure Virtual Machines de un conjunto de disponibilidad de Azure en diferentes dominios de error y de actualización. El propósito de la distribución en diferentes dominios de error y de actualización es evitar el cierre de todas las máquinas virtuales de un sistema SAP en caso de mantenimiento de la infraestructura o error en un dominio de error. De forma predeterminada, las máquinas virtuales no forman parte de un conjunto de disponibilidad. La participación de una máquina virtual en un conjunto de disponibilidad se define en el momento de la implementación o más adelante mediante una reconfiguración y reimplementación de una máquina virtual.

Para comprender el concepto de conjuntos de disponibilidad de Azure y la relación entre los conjuntos de disponibilidad y los dominios de error y actualización, lea [este artículo][virtual-machines-manage-availability]. 

A medida que se vayan definiendo conjuntos de disponibilidad y se intenten combinar varias máquinas virtuales de distintas familias de máquinas virtuales en un mismo conjunto de disponibilidad, es posible que se produzcan problemas que impidan incluir un determinado tipo de máquina virtual en dicho conjunto de disponibilidad. El motivo es que el conjunto de disponibilidad está enlazado a una unidad de escalado que contiene un determinado tipo de hosts de proceso, y un tipo de host de proceso determinado solo puede ejecutar ciertos tipos de familias de máquinas virtuales. Por ejemplo, si crea un conjunto de disponibilidad e implementa la primera máquina virtual en él con un tipo de máquina virtual de la familia Esv3, si luego intenta implementar una segunda máquina virtual de la familia M, la segunda asignación se rechazará. La razón es que las máquinas virtuales de la familia Esv3 no se ejecutan en el mismo hardware de host que las máquinas virtuales de la familia M. Este mismo problema puede suceder si se intenta cambiar el tamaño de las máquinas virtuales y se intenta trasladar una máquina virtual de la familia Esv3 a un tipo de máquina virtual de la familia M. Si se cambia el tamaño al de una familia de máquinas virtuales que no se puede hospedar en el mismo hardware de host, deberá apagar todas las máquinas virtuales del conjunto de disponibilidad y cambiarlas de tamaño para que puedan ejecutarse en el otro tipo de máquina host. Para conocer las particularidades de los SLA que hay implementados en un conjunto de disponibilidad, vea el artículo [SLA para máquinas virtuales](https://azure.microsoft.com/support/legal/sla/virtual-machines/). 

Los principios de conjunto de disponibilidad y de dominios de actualización y de error relacionados no son válidos en el servicio específico de HANA de [instancias grandes de HANA](./hana-overview-architecture.md). Encontrará los Acuerdos de Nivel de Servicio del servicio de instancias grandes de HANA en el artículo [SLA para SAP HANA e instancias grandes de Azure](https://azure.microsoft.com/support/legal/sla/sap-hana-large/). 

> [!IMPORTANT]
> Los conceptos de zonas de disponibilidad de Azure y de conjuntos de disponibilidad de Azure son mutuamente excluyentes. Esto significa que se pueden implementar dos o varias máquinas virtuales en una zona de disponibilidad específica o en un conjunto de disponibilidad de Azure, pero no en ambos.

### <a name="azure-paired-regions"></a>Regiones emparejadas de Azure
Azure ofrece pares de regiones de Azure concretas en que la replicación de determinados datos está habilitada. El emparejamiento de regiones se documenta en el artículo [Continuidad empresarial y recuperación ante desastres (BCDR): Regiones emparejadas de Azure](../../../best-practices-availability-paired-regions.md). Como se describe en el artículo, la replicación de datos está ligada a tipos de almacenamiento de Azure que pueden configurar para que se repliquen en la región emparejada. Consulte también el artículo [Redundancia en una región secundaria](../../../storage/common/storage-redundancy.md#redundancy-in-a-secondary-region). Los tipos de almacenamiento que permiten esa replicación son tipos de almacenamiento que no son adecuados para cargas de trabajo de DBMS. Como tal, la facilidad de uso de la replicación de Azure Storage se limitaría a Azure Blob Storage (por ejemplo, con fines de copia de seguridad) u otros escenarios de almacenamiento de alta latencia. A medida que busque regiones emparejadas y los servicios que quiera usar como región principal o secundaria, puede que se encuentre en situaciones en las que los servicios de Azure o los tipos de VM que quiere usar en la región principal no estén disponibles en la región emparejada. O puede encontrarse con una situación en la que la región emparejada de Azure no sea aceptable por motivos de cumplimiento de datos. En esos casos, debe usar una región no emparejada como región de recuperación ante desastres o secundaria. Para ese tipo de situaciones, debe tener cuidado en la replicación de parte de los datos en que Azure le habría replicado. En el artículo [Configuración de la recuperación ante desastres para Active Directory y DNS](../../../site-recovery/site-recovery-active-directory.md), se ofrece un ejemplo acerca de cómo replicar la instancia de Active Directory y el DNS en la región de la recuperación ante desastres.
 

## <a name="azure-virtual-machine-services"></a>Servicios de máquinas virtuales de Azure
Azure ofrece una gran variedad de máquinas virtuales que puede seleccionar para implementarlas. No existe necesidad de realizar adquisiciones iniciales de tecnología e infraestructura. El servicio de máquinas virtuales de Azure simplifica el mantenimiento y funcionamiento de aplicaciones, proporcionando procesos y almacenamiento a petición para hospedar, escalar y administrar aplicaciones web y aplicaciones conectadas. La administración de la infraestructura se automatiza con una plataforma diseñada para lograr alta disponibilidad y escalado dinámico con el fin de satisfacer las necesidades de uso con la posibilidad de optar entre diferentes modelos de precios.

![Posicionamiento de servicios de máquinas virtuales de Microsoft Azure][planning-guide-figure-400]

Con los servicios de máquina virtual de Azure, Microsoft permite implementar imágenes de servidor personalizadas en Azure como instancias de IaaS. También puede elegir entre una amplia selección de imágenes de sistema operativo consumibles de la galería de imágenes de Azure.

Desde una perspectiva operativa, el servicio de máquinas virtuales de Azure es similar a las máquinas virtuales implementadas en local. El usuario es el responsable de la administración, de las operaciones y también de la aplicación de revisiones del sistema operativo concreto que se ejecuta en una máquina virtual de Azure y de las aplicaciones en esa máquina virtual. El único servicio que Microsoft presta es el hospedaje de esa máquina virtual en su infraestructura de Azure (infraestructura como servicio, o IaaS). Microsoft no tiene ofertas, aparte de las relativas a IaaS, para las cargas de trabajo de SAP que un usuario implemente como cliente. 

La plataforma Microsoft Azure es una plataforma multiempresa. Esto quiere decir que los recursos de almacenamiento, red y proceso que hospedan máquinas virtuales de Azure se comparten entre los inquilinos, con algunas excepciones. Se usa la limitación inteligente y la lógica de cuotas para impedir que un inquilino perjudique el rendimiento de otro (vecino ruidoso) significativamente. Concretamente para certificar la plataforma Azure para SAP HANA, Microsoft debe demostrar el aislamiento de recursos para los casos donde se pueden ejecutar varias máquinas virtuales en el mismo host de forma periódica para SAP. Aunque la lógica de Azure intenta que las variaciones de ancho de banda sean pequeñas, las plataformas con más uso compartido suelen generar mayores variaciones en la disponibilidad de recursos y ancho de banda que las que puedan experimentar los clientes en sus implementaciones locales. Debe tenerse en cuenta que un sistema SAP en Azure podría sufrir mayores variaciones que un entorno local.

### <a name="azure-virtual-machines-for-sap-workload"></a>Máquinas virtuales de Azure para la carga de trabajo de SAP

Respecto a las cargas de trabajo de SAP, hemos reducido la selección a distintas familias de máquinas virtuales que son adecuadas para las cargas de trabajo de SAP y de SAP HANA de un modo más específico. La forma en que se puede encontrar el tipo de máquina virtual correcto y su capacidad para funcionar con una carga de trabajo de SAP se describe en el documento [Qué software de SAP es compatible para su implementación en Azure](./sap-supported-product-on-azure.md). 

> [!NOTE]
> En los tipos de máquina virtual certificados para la carga de trabajo de SAP no hay ningún exceso de aprovisionamiento de los recursos de CPU y de memoria.

Aparte de seleccionar los tipos de VM que son compatibles, también hay que comprobar si están disponibles en una región específica según se describe en [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/). Pero lo más importante es evaluar los siguientes aspectos:

- Recursos de CPU y memoria de diferentes tipos de máquina virtual 
- Ancho de banda de IOPS de distintos tipos de máquina virtual
- Funcionalidades de red de distintos tipos de máquina virtual
- Número de discos que se pueden conectar
- Capacidad de aprovechar algunos tipos de almacenamiento de Azure

La mayoría de los datos relativos a un tipo de máquina virtual concreto se puede encontrar [aquí (Linux)][virtual-machines-sizes-linux] y [aquí (Windows)][virtual-machines-sizes-windows].

Como modelo de precios, existen varias opciones de precios diferentes, como las siguientes:

- Pago por uso
- Un año reservado
- Tres años reservados
- Precios al contado

Los precios de cada una de las ofertas, con diferentes ofertas de servicio relativas a distintos sistemas operativos y regiones, están disponibles en los sitios [Precios de máquinas virtuales Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) y [Precios de máquinas virtuales Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). Vea estos artículos para más información y conocer la flexibilidad de las instancias de reserva de uno y tres años:

- [¿Qué es Azure Reservations?](../../../cost-management-billing/reservations/save-compute-costs-reservations.md)
- [Flexibilidad en el tamaño de las máquinas virtuales con Azure Reserved VM Instances](../../windows/reserved-vm-instance-size-flexibility.md)
- [Aplicación del descuento por reserva de Azure en las máquinas virtuales](../../../cost-management-billing/manage/understand-vm-reservation-charges.md) 

Para más información sobre los precios al contado, lea el artículo [Máquinas virtuales de Azure al contado](https://azure.microsoft.com/pricing/spot/). Los precios del mismo tipo de máquina virtual también pueden ser diferentes en cada región de Azure. Para algunos clientes, merecerá más la pena implementar en una región de Azure menos costosa.

Azure ofrece además el concepto de host dedicado. El concepto de host dedicado proporciona más control sobre los ciclos de revisión que Azure realiza. Puede ajustar la aplicación de revisiones a sus programaciones personales propias. Esta oferta va dirigida específicamente a clientes con cargas de trabajo que pueden desviarse del ciclo normal de carga de trabajo. Para ahondar en los conceptos de ofertas de host dedicado de Azure, lea el artículo [Hosts dedicados de Azure](../../windows/dedicated-hosts.md). Esta oferta se puede usar con la carga de trabajo de SAP, y la emplean varios clientes de SAP que quieren tener más control sobre la aplicación de revisiones en la infraestructura y en los planes de mantenimiento finales de Microsoft. Para más información sobre cómo Microsoft mantiene y revisa la infraestructura de Azure que hospeda máquinas virtuales, lea el artículo [Mantenimiento de máquinas virtuales en Azure](../../maintenance-and-updates.md).

#### <a name="generation-1-and-generation-2-virtual-machines"></a>Máquinas virtuales de generación 1 y generación 2
El hipervisor de Microsoft es capaz de administrar dos generaciones diferentes de máquinas virtuales. Estos formatos se denominan **generación 1** y **generación 2**. La **generación 2** se presentó en el año 2012 con el hipervisor de Windows Server 2012. Azure empezó usando máquinas virtuales de generación 1. Cuando implemente máquinas virtuales de Azure, el comportamiento predeterminado es seguir usando el formato de la generación 1. Mientras tanto, también puede implementar formatos de máquina virtual de generación 2. En el artículo [Compatibilidad para máquinas virtuales de generación 2 en Azure](../../windows/generation-2.md) se enumeran las familias de máquinas virtuales de Azure que se pueden implementar como máquinas virtuales de generación 2. En este artículo también se describen las importantes diferencias funcionales de las máquinas virtuales de generación 2, ya que se pueden ejecutar en la nube privada de Hyper-V y Azure. Más importante aún, en este artículo también se explican las diferencias funcionales entre las máquinas virtuales de generación 1 y las máquinas virtuales de generación 2 que se ejecutan en Azure. 

> [!NOTE]
> Existen diferencias funcionales entre las máquinas virtuales de generación 1 y de generación 2 que se ejecutan en Azure. Lea el artículo [Compatibilidad para máquinas virtuales de generación 2 en Azure](../../windows/generation-2.md) para conocer esas diferencias.  
 
Una máquina virtual existente no se puede mover de una generación a otra. Para cambiar la generación de una máquina virtual, deberá implementar una nueva VM de la generación que quiera y reinstalar el software que se esté ejecutando en la máquina virtual de la generación en cuestión. Este cambio solo afecta a la imagen de VHD base de la VM, y no tiene impacto alguno en los discos de datos o en los recursos compartidos NFS o SMB conectados. Los discos de datos o los recursos compartidos NFS o SMB que se asignaron inicialmente, por ejemplo, en una VM de generación 1. 

> [!NOTE]
> A partir del 1 de mayo de 2020, es posible implementar máquinas virtuales de la familia Mv1 como máquinas virtuales de generación 2. Gracias a esto, podrán realizarse aumentos y reducciones de tamaño sin fisuras entre las máquinas virtuales de la familia Mv1 y MV2.
 

### <a name="storage-microsoft-azure-storage-and-data-disks"></a><a name="a72afa26-4bf4-4a25-8cf7-855d6032157f"></a>Storage: Microsoft Azure Storage y discos de datos
Microsoft Azure Virtual Machines usa varios tipos de almacenamiento. Al implementar SAP en los servicios de máquina virtual de Azure, es importante comprender las diferencias entre estos dos tipos principales de almacenamiento:

* Almacenamiento volátil, no persistente.
* Almacenamiento persistente.

Las máquinas virtuales de Azure ofrecen discos no persistentes después de implementar una máquina virtual. En el caso de un reinicio de máquina virtual, se borrará todo el contenido de esas unidades. Por tanto, es evidente que los archivos de datos y los archivos de registro y fase de puesta al día de las bases de datos no se deben colocar bajo ninguna circunstancia en esas unidades de disco no persistentes. Es posible que haya excepciones para algunas de las bases de datos, donde estas unidades de disco no persistentes podrían ser adecuadas para tempdb y los espacios de tablas temporales. Pero evite usar esas unidades de disco para las máquinas virtuales de la serie A, ya que esas unidades no persistentes tienen un límite de rendimiento con esa familia de máquinas virtuales. Para obtener más información, lea el artículo [Understanding the temporary drive on Windows VMs in Azure](/archive/blogs/mast/understanding-the-temporary-drive-on-windows-azure-virtual-machines) (Descripción de la unidad temporal en máquinas virtuales de Windows en Azure).

---
> ![Windows][Logo_Windows] Windows
> 
> La unidad D:\ de una máquina virtual de Azure es una unidad no persistente respaldada por algunos discos locales del nodo de proceso de Azure. Como no es persistente, significa que los cambios realizados en el contenido de la unidad D:\ se perderán cuando se reinicie la máquina virtual. Por "cambios", se refiere a los archivos guardados, los directorios creados, las aplicaciones instaladas, etc.
> 
> ![Linux][Logo_Linux] Linux
> 
> Las máquinas virtuales de Linux de Azure montan automáticamente una unidad en /mnt/resource, que es una unidad no persistente respaldada por discos locales del nodo de proceso de Azure. Como no es persistente, significa que los cambios realizados en el contenido de la unidad /mnt/resource se perderán cuando se reinicie la máquina virtual. Por cambios, se refiere a los archivos guardados, los directorios creados, las aplicaciones instaladas, etc.
> 
> 

#### <a name="azure-storage-accounts"></a>Cuentas de Azure Storage

Al implementar servicios o VM en Azure, la implementación de discos duros virtuales e imágenes de VM se organiza en unidades denominadas cuentas de almacenamiento de Azure. [Las cuentas de almacenamiento de Azure](../../../storage/common/storage-account-overview.md) tienen limitaciones respecto a las IOPS, el rendimiento o los tamaños que pueden contener. En el pasado, estas limitaciones, que se documentan en: 

- [Objetivos de escalabilidad para cuentas de almacenamiento estándar](../../../storage/common/scalability-targets-standard-account.md)
- [Objetivos de escalabilidad de las cuentas de almacenamiento de blob en páginas Premium](../../../storage/blobs/scalability-targets-premium-page-blobs.md)

tuvieron un rol importante en la planeación de una implementación de SAP en Azure. Tenía que administrar el número de discos persistentes en una cuenta de almacenamiento. Necesitaba administrar las cuentas de almacenamiento y, finalmente, crear nuevas cuentas de almacenamiento para crear más discos persistentes. 

En los últimos años, la introducción de [Azure Managed Disks](../../windows/managed-disks-overview.md) le ha liberado de esas tareas. La recomendación para las implementaciones de SAP es aprovechar Azure Managed Disks en lugar de administrar las cuentas de almacenamiento de Azure manualmente. Azure Managed Disks distribuirá discos entre diferentes cuentas de almacenamiento de modo que no se superen los límites de cada cuenta.

En una cuenta de almacenamiento, tiene un tipo de concepto de carpeta denominado "contenedores" que se puede usar para agrupar determinados discos en contenedores específicos.

Dentro de Azure, un nombre de disco o VHD tiene la siguiente nomenclatura, que debe indicar un nombre único para el VHD en Azure:

`http(s)://<storage account name>.blob.core.windows.net/<container name>/<vhd name>`

La cadena de texto anterior tiene que identificar de forma unívoca el disco o VHD almacenado en Azure Storage.


#### <a name="azure-persisted-storage-types"></a>Tipos de almacenamiento persistente de Azure
Azure ofrece una variedad de opciones de almacenamiento persistentes que se pueden usar para los componentes de carga de trabajo de SAP y pila de SAP específica. Para obtener más información, lea el documento [Azure Storage para cargas de trabajo de SAP](./planning-guide-storage.md).


### <a name="microsoft-azure-networking"></a><a name="61678387-8868-435d-9f8c-450b2424f5bd"></a>Redes de Microsoft Azure

Microsoft Azure proporciona una infraestructura de red que permite la correspondencia de todos los escenarios que deseamos obtener con el software SAP. Principales capacidades:

* Acceso desde el exterior directamente a las máquinas virtuales a través de ssh/VNC o de Windows Terminal Services
* Acceso a servicios y puertos específicos usados por las aplicaciones dentro de las máquinas virtuales
* Comunicación interna y resolución de nombres entre un grupo de máquinas virtuales implementadas como máquinas virtuales de Azure
* Conectividad entre locales entre la red local de un cliente y la red de Azure
* Conectividad entre sitios de Azure y varias regiones de Azure o varios centros de datos

Puede encontrar más información aquí: <https://azure.microsoft.com/documentation/services/virtual-network/>

Hay muchas posibilidades diferentes para configurar el nombre y la resolución IP en Azure. También puede usarse un servicio Azure DNS en lugar de configurar un servidor DNS propio. Se puede encontrar más información en [este artículo][virtual-networks-manage-dns-in-vnet] y en [esta página](https://azure.microsoft.com/services/dns/).

Para escenarios entre locales o híbridos, se da por hecho que las instalaciones locales de AD/OpenLDAP/DNS se han ampliado a través de VPN o con una conexión privada a Azure. Para determinados escenarios documentados aquí, sería necesario disponer de una réplica de AD/OpenLDAP instalada en Azure.

Dado que la red y la resolución de nombres es una parte fundamental de la implementación de bases de datos en un sistema SAP, este concepto se analiza con más detenimiento en la [Guía de implementación de DBMS][dbms-guide].

##### <a name="azure-virtual-networks"></a>Instancias de Azure Virtual Network

Se puede definir el intervalo de direcciones IP privadas asignadas por la funcionalidad de DHCP de Azure mediante la creación de una instancia de Azure Virtual Network. En escenarios entre locales, Azure seguirá asignando el intervalo de direcciones IP definido mediante DHCP. Sin embargo, la resolución de nombres de dominio se lleva a cabo en local (suponiendo que las máquinas virtuales formen parte de un dominio local) y, por tanto, se pueden resolver direcciones independientemente de los diferentes servicios de Azure Cloud Services.

Cada máquina virtual de Azure debe estar conectada a una red virtual.

Se puede encontrar más información en [este artículo][resource-groups-networking] y en [esta página](https://azure.microsoft.com/documentation/services/virtual-network/).


> [!NOTE]
> De forma predeterminada, una vez que se implementa una máquina virtual no se puede cambiar la configuración de red virtual. La configuración de TCP/IP debe depender del servidor DHCP de Azure. El comportamiento predeterminado es la asignación de IP dinámicas.
>
>

La dirección MAC de la tarjeta de red virtual puede cambiar, por ejemplo, después de un cambio de tamaño. En ese caso, el SO invitado Windows o Linux identifica la nueva tarjeta de red y utiliza automáticamente DHCP para asignar las direcciones IP y DNS.

##### <a name="static-ip-assignment"></a>Asignación de direcciones IP estáticas
Se pueden asignar direcciones IP reservadas o fijas a las máquinas virtuales de una instancia de Azure Virtual Network. La ejecución de máquinas virtuales en una instancia de Azure Virtual Network es una gran oportunidad para aprovechar esta funcionalidad así lo requieren algunos escenarios. La asignación de IP seguirá siendo válida mientras exista la máquina virtual, independientemente de si la máquina virtual está en ejecución o cerrada. Como resultado, se tiene que tener en cuenta el número total de máquinas virtuales (en ejecución y detenidas) al definir el intervalo de direcciones IP para Virtual Network. La asignación de dirección IP se mantiene hasta que se elimine la máquina virtual y su interfaz de red o hasta que se cancele la asignación. Para más información, consulte [este artículo][virtual-networks-static-private-ip-arm-pportal].

> [!NOTE]
> Debería asignar direcciones IP estáticas mediante Azure a NIC virtuales individuales. No debería asignar direcciones IP estáticas dentro del sistema operativo invitado a una NIC virtual. Algunos servicios de Azure como el servicio Azure Backup se basan en el hecho de que al menos el vNIC principal está establecido en DHCP y no en direcciones IP estáticas. Consulte también el documento [Solución de problemas de copia de seguridad de máquinas virtuales de Azure](../../../backup/backup-azure-vms-troubleshoot.md#networking).
>
>

##### <a name="multiple-nics-per-vm"></a>Varias NIC por máquina virtual

Se pueden definir varias tarjetas de interfaz de red virtual (vNIC) en una máquina virtual de Azure. Con la capacidad de tener varias vNIC, se puede comenzar a configurar el tráfico de red donde, por ejemplo, el tráfico del cliente se enrute a través de una vNIC y el tráfico back-end se enrute a través de una segunda vNIC. En función del tipo de máquina virtual, hay varias limitaciones con respecto al número de vNIC que puede tener asignada una máquina virtual. En los artículos siguientes se indican los detalles, las funcionalidades y las restricciones:

* [Creación de una máquina virtual Windows con varias tarjetas de interfaz de red][virtual-networks-multiple-nics-windows]
* [Creación de una máquina virtual Linux con varias NIC][virtual-networks-multiple-nics-linux]
* [Implementación de varias máquinas virtuales de NIC con una plantilla][virtual-network-deploy-multinic-arm-template]
* [Implementación de máquinas virtuales con varias NIC mediante PowerShell][virtual-network-deploy-multinic-arm-ps]
* [Implementación de varias máquinas virtuales de la NIC con la CLI de Azure][virtual-network-deploy-multinic-arm-cli]

#### <a name="site-to-site-connectivity"></a>Conectividad de sitio a sitio

Las máquinas virtuales de Azure tienen características entre locales, mientras que las instalaciones locales están vinculadas a una conexión VPN permanente y transparente. Se espera que se convierta en el modelo de implementación de SAP más común en Azure. Se supone que los procesos y procedimientos operativos con instancias de SAP en Azure deben funcionar de forma transparente. Esto significa que se podrá imprimir a partir de estos sistemas, así como el sistema de administración del transporte (TMS) de SAP para el transporte de cambios de un sistema de desarrollo en Azure a un sistema de prueba de implementación local. Se puede encontrar más documentación sobre la conectividad de sitio a sitio en [este artículo][vpn-gateway-create-site-to-site-rm-powershell]

##### <a name="vpn-tunnel-device"></a>Dispositivo de túnel VPN

Para crear una conexión de sitio a sitio (centro de datos local al centro de datos de Azure), necesita obtener o configurar un dispositivo VPN, o utilizar el servicio de Enrutamiento y acceso remoto (RRAS) que se introdujo como componente de software en Windows Server 2012.

* [Create a virtual network with a site-to-site VPN connection using PowerShell][vpn-gateway-create-site-to-site-rm-powershell] (Creación de una red virtual con conexión VPN de sitio a sitio mediante PowerShell)
* [Acerca de los dispositivos VPN para las conexiones de VPN Gateway de sitio a sitio][vpn-gateway-about-vpn-devices]
* [Preguntas más frecuentes sobre VPN Gateway][vpn-gateway-vpn-faq]

![Conexión de sitio a sitio entre sistemas locales y Azure][planning-guide-figure-600]

La ilustración anterior muestra dos suscripciones de Azure con subintervalos de direcciones IP reservados para las redes virtuales en Azure. La conectividad desde la red local a Azure se establece a través de VPN.

#### <a name="point-to-site-vpn"></a>VPN de punto a sitio

VPN de punto a sitio requiere que cada equipo cliente se conecte a Azure con su propio VPN. En los escenarios SAP examinados aquí, la conectividad de punto a sitio no es una solución práctica. Por tanto, no se proporcionan más referencias de la conectividad VPN punto a sitio.

Puede encontrar más información aquí.
* [Configuración de una conexión de punto a sitio a una red virtual mediante Azure Portal](../../../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md)
* [Configuración de una conexión de punto a sitio a una red virtual mediante PowerShell](../../../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)

#### <a name="multi-site-vpn"></a>VPN de varios sitios

Asimismo, en la actualidad Azure ofrece la posibilidad de crear una conectividad VPN multisitio para una suscripción de Azure. Anteriormente, las suscripciones únicas estaban limitaban a una conexión VPN de sitio a sitio. Esta limitación desapareció con las conexiones VPN multisitio para suscripciones únicas. De este modo, una suscripción específica puede aprovechar varias regiones de Azure mediante configuraciones entre locales.

Para más documentación, consulte [este artículo][vpn-gateway-create-site-to-site-rm-powershell]

#### <a name="vnet-to-vnet-connection"></a>Conexión de red virtual a red virtual

Con VPN multisitio, se debe configurar una instancia de Azure Virtual Network independiente en cada región. De todas formas, a menudo se requiere que los componentes de software de las diferentes regiones se comuniquen entre sí. Lo ideal sería que esta comunicación no se enrutara desde una región de Azure a instalaciones locales, y desde ahí a otra región de Azure. Para el acceso directo, Azure ofrece la posibilidad de configurar una conexión de una instancia de Azure Virtual Network en una región a otra instancia de Azure Virtual Network hospedada en otra región. Esta funcionalidad se denomina conexión de red virtual a red virtual. Puede encontrar más información sobre esta funcionalidad aquí: <https://azure.microsoft.com/documentation/articles/vpn-gateway-vnet-vnet-rm-ps/>.

#### <a name="private-connection-to-azure-expressroute"></a>Conexión privada a Azure ExpressRoute

Microsoft Azure ExpressRoute permite la creación de conexiones privadas entre centros de datos de Azure y la infraestructura local del cliente o un entorno de colocalización. ExpressRoute está disponible en varios proveedores de VPN de MPLS (conmutación por paquetes) u otros proveedores de servicios de red. Las conexiones ExpressRoute no pasan por la red pública de Internet. Las conexiones ExpressRoute ofrecen más confiabilidad, velocidad y seguridad en varios circuitos en paralelo, además de mayor velocidad y menor latencia que las conexiones a Internet habituales.

Se puede obtener más información sobre Azure ExpressRoute y su oferta aquí:

* <https://azure.microsoft.com/documentation/services/expressroute/>
* <https://azure.microsoft.com/pricing/details/expressroute/>
* <https://azure.microsoft.com/documentation/articles/expressroute-faqs/>

ExpressRoute permite varias suscripciones de Azure a través de un circuito ExpressRoute, como está documentado aquí

* <https://azure.microsoft.com/documentation/articles/expressroute-howto-linkvnet-arm/>
* <https://azure.microsoft.com/documentation/articles/expressroute-howto-circuit-arm/>

#### <a name="forced-tunneling-in-case-of-cross-premises"></a>Tunelización forzada en instalaciones entre locales
Con las máquinas virtuales que unan dominios locales a través de conexiones sitio a sitio, punto a sitio o de ExpressRoute, también hay que asegurarse de implementar la configuración de proxy de Internet para todos los usuarios de esas máquinas virtuales. De forma predeterminada, el software ejecutado en esas máquinas virtuales y los usuarios que empleen un navegador se conectarán directamente a Internet a través de Azure, en lugar de a través del proxy de la empresa. Aun así, la configuración de proxy no es una solución completa para dirigir el tráfico a través del proxy de la empresa, ya que es responsabilidad del software y los servicios buscar el servidor proxy. Si el software ejecutado en la máquina virtual no lleva a cabo esta operación o un administrador modifica la configuración, el tráfico a Internet podría desviarse de nuevo directamente a través de Azure.

Para evitar esta conectividad directa a Internet, puede configurar la tunelización forzada con conectividad de sitio a sitio entre instalaciones locales y Azure. La descripción detallada de la característica de tunelización forzada se publica aquí: <https://azure.microsoft.com/documentation/articles/vpn-gateway-forced-tunneling-rm/>

La tunelización forzada con ExpressRoute la habilitan clientes que anuncian una ruta predeterminada a través de las sesiones de emparejamiento BGP de ExpressRoute.

#### <a name="summary-of-azure-networking"></a>Resumen de las redes de Azure

Este capítulo contiene muchos puntos importantes sobre la red de Azure. Este es un resumen de los puntos principales:

* Azure Virtual Network permite colocar una estructura de red en la implementación de Azure. Las redes virtuales se pueden aislar unas de otras, o con la ayuda de los grupos de seguridad de red se puede controlar el tráfico entre redes virtuales.
* Las instancias de Azure Virtual Network pueden aprovecharse para asignar intervalos de direcciones IP a las máquinas virtuales o asignar direcciones IP fijas a máquinas virtuales.
* Para configurar una conexión de sitio a sitio o de punto a sitio, primero se debe crear una instancia de Azure Virtual Network.
* Una vez que se ha implementado una máquina virtual, ya no se puede cambiar la red virtual asignada a la máquina virtual.

### <a name="quotas-in-azure-virtual-machine-services"></a>Cuotas de los servicios de máquinas virtuales de Azure
Debe quedar claro que la infraestructura de red y almacenamiento se comparte entre las máquinas virtuales que ejecuten una amplia variedad de servicios en la infraestructura de Azure. Al igual que en los centros de datos del cliente, hay un cierto aprovisionamiento excesivo de algunos de los recursos de la infraestructura. La plataforma Microsoft Azure utiliza cuotas de disco, de CPU, de red y otras cuotas para limitar el consumo de recursos y mantener un rendimiento uniforme y determinista.  Los diferentes tipos de máquinas virtuales (A5, A6, etc.) tienen diferentes cuotas de número de discos, CPU, RAM y red.

> [!NOTE]
> Los recursos de memoria y CPU de los tipos de máquina virtual compatibles con SAP se preasignan en los nodos de host. Esto significa que, una vez implementada la máquina virtual, los recursos en el host están disponibles según defina el tipo de máquina virtual.
>
>

Al planear y dimensionar SAP en las soluciones de Azure, se tienen que tener en cuenta las cuotas de tamaño de cada máquina virtual. Las cuotas de máquina virtual se describen [aquí (Linux)][virtual-machines-sizes-linux] y [aquí (Windows)][virtual-machines-sizes-windows].

Las cuotas descritas representan los valores máximos teóricos.  El límite de E/S por segundo por disco se puede lograr con valores de E/S pequeños (8 KB), pero, posiblemente, no pueda lograrse con valores de E/S grandes (1 MB).  El límite de E/S por segundo se aplica en la granularidad de discos únicos.

Es posible utilizar el siguiente árbol de decisión aproximado para decidir si un sistema SAP encaja en los servicios de máquinas virtuales de Azure y sus funciones, o bien si un sistema existente debe configurarse de otro modo para poder implementar el sistema en Azure:

![Árbol de decisión para decidir la capacidad de implementación de SAP en Azure][planning-guide-figure-700]

1. La información más importante por la que empezar es el requisito de SAPS de un sistema SAP determinado. Los requisitos de SAPS deben dividirse en la parte de DBMS y en la de la aplicación SAP, aunque el sistema SAP ya esté implementado de forma local en una configuración de dos niveles. Para los sistemas existentes, los SAPS relacionados con el hardware que se usa se suele poder determinar o calcular mediante los bancos de pruebas de SAP existentes. Los resultados se pueden encontrar [aquí](https://sap.com/about/benchmark.html). En lo que respecta a los sistemas SAP de implementación reciente, tendrá que haber efectuado una tarea de determinación del tamaño con la que se habrán identificado los requisitos de SAPS del sistema. 
1. Para los sistemas existentes, se debe medir el volumen de E/S y las operaciones de E/S por segundo en el servidor de DBMS. En el caso de los sistemas planeados recientemente, la tarea de determinación de tamaño para el nuevo sistema también debe aportar una idea aproximada de los requisitos de E/S para DBMS. Si no está seguro, tendrá que terminar realizando una prueba de concepto.
1. Compare el requisito de SAPS del servidor de DBMS con los SAPS que pueden proporcionar los distintos tipos de máquinas virtuales de Azure. La información sobre los SAPS de los diferentes tipos de máquinas virtuales de Azure se incluye en la nota de SAP [1928533]. Primero hay que centrarse en la máquina virtual de DBMS, puesto que la de la base de datos es la capa de un sistema SAP NetWeaver que no se escala horizontalmente en la mayoría de las implementaciones. Por el contrario, la capa de aplicaciones de SAP se puede escalar horizontalmente. Si ninguno de los tipos de máquinas virtuales de Azure compatibles con SAP puede proporcionar los SAP necesarios, la carga de trabajo del sistema SAP planeado no podrá ejecutarse en Azure. Tendrá que implementar el sistema de forma local o cambiar su volumen de carga de trabajo.
1. Como se documenta [aquí (Linux)][virtual-machines-sizes-linux] y [aquí (Windows)][virtual-machines-sizes-windows], Azure exige una cuota de IOPS para cada disco, al margen de si se usa Standard Storage o Premium Storage. El número de discos de datos que se pueden montar varía según el tipo de máquina virtual. Como resultado, puede calcular el número máximo de IOPS que se puede lograr con cada uno de los distintos tipos de máquinas virtuales. Según el diseño del archivo de base de datos, puede seccionar los discos para convertirlos en un volumen en el SO invitado. Sin embargo, si el volumen actual de IOPS de un sistema SAP implementado supera los límites calculados del tipo de máquina virtual más grande de Azure y no hay ninguna posibilidad de compensar con más memoria, la carga de trabajo del sistema SAP puede verse gravemente afectada. En tales casos, se puede llegar a un punto en el que no se debería implementar el sistema en Azure.
1. Especialmente en sistemas SAP que estén implementados localmente en configuraciones de dos niveles, lo más probable es que el sistema deba configurarse en Azure en una configuración de tres niveles. En este paso, debe comprobar si hay algún componente en la capa de aplicaciones de SAP que no se pueda escalar horizontalmente y que no podría encajar con los recursos de memoria y CPU que ofrecen los distintos tipos de máquinas virtuales de Azure. Si, en efecto, se detecta tal componente, no se podrá implementar en Azure el sistema SAP ni su carga de trabajo. Pero, si es posible escalar horizontalmente los componentes de las aplicaciones de SAP en varias máquinas virtuales de Azure, el sistema puede implementarse en Azure.

Si los componentes de capa de la aplicación de SAP y DBMS pueden ejecutarse en máquinas virtuales de Azure, se debe definir una serie de opciones de configuración relacionadas con los siguientes aspectos:

* Número de máquinas virtuales de Azure
* Tipos de máquinas virtuales para los componentes individuales
* Número de VHD en la máquina virtual de DBMS para proporcionar IOPS suficientes

## <a name="managing-azure-assets"></a>Administración de activos de Azure

### <a name="azure-portal"></a>Azure portal

Azure Portal constituye una de las tres interfaces creadas para administrar las implementaciones de máquinas virtuales de Azure. Es posible realizar las tareas de administración básicas, como implementar máquinas virtuales a partir de imágenes, a través de Azure Portal. Además, la creación de cuentas de almacenamiento, redes virtuales y otros componentes de Azure son tareas que pueden desempeñarse con resultados satisfactorios en Azure Portal. No obstante, funciones como la carga de VHD de entornos locales a Azure o la copia de un VHD dentro de Azure constituyen tareas para las que se requiere bien una herramienta de terceros, bien conocimientos de administración a través de PowerShell o la CLI.

![Microsoft Azure Portal: información general sobre las máquinas virtuales][planning-guide-figure-800]


También es posible llevar a cabo tareas de administración y configuración para las instancias de las máquinas virtuales desde Azure Portal.

Aparte de reiniciar y apagar una máquina virtual, también puede asociar, desasociar y crear discos de datos para la instancia de Virtual Machine con el objetivo de capturarla para preparar una imagen y configurar el tamaño de dicha instancia.

Azure Portal proporciona funciones básicas con las que implementar y configurar las máquinas virtuales y muchos otros servicios de Azure. Sin embargo, Azure Portal no abarca todas las funciones disponibles. En Azure Portal, no es posible realizar tareas como las siguientes:

* Cargar discos duros virtuales en Azure
* Copiar máquinas virtuales


### <a name="management-via-microsoft-azure-powershell-cmdlets"></a>Administración mediante los cmdlets de Microsoft Azure PowerShell

Windows PowerShell es un marco eficaz y extensible que ha tenido una amplia adopción entre clientes que implementan un mayor número de sistemas en Azure. Es posible ejecutar de forma remota los cmdlets de PowerShell tras instalarlos en un equipo de escritorio, un portátil o una estación de administración dedicada.

En [este artículo][powershell-install-configure] se describe el proceso mediante el cual se habilita el uso de cmdlets de Azure PowerShell en un equipo de escritorio o portátil local, así como el modo de configurarlos para su uso con las suscripciones de Azure.

También podrá encontrar pasos más detallados sobre cómo instalar, actualizar y configurar los cmdlets de Azure PowerShell en [este capítulo de la Guía de implementación][deployment-guide-4.1].

Hasta el momento, de la experiencia de los clientes se extrae que PowerShell (PS) es sin lugar a dudas la herramienta más eficaz para implementar máquinas virtuales y crear pasos personalizados en la implementación de estas. Todos los clientes que ejecutan instancias de SAP en Azure usan cmdlets de PS como complemento a las tareas de administración que llevan a cabo en Azure Portal o incluso los utilizan exclusivamente para administrar sus implementaciones en Azure. Como los cmdlets específicos de Azure comparten la misma convención de nomenclatura que los más de 2000 cmdlets relacionados con Windows, a los administradores de Windows les resulta sencillo utilizarlos.

Consulte el ejemplo a continuación: <https://blogs.technet.com/b/keithmayer/archive/2015/07/07/18-steps-for-end-to-end-iaas-provisioning-in-the-cloud-with-azure-resource-manager-arm-powershell-and-desired-state-configuration-dsc.aspx>


La implementación de la extensión de Azure para SAP (consulte el capítulo [Extensión de Azure para SAP][planning-guide-9.1] de este documento) solo es posible mediante PowerShell o la CLI. Por lo tanto, es obligatorio para instalar y configurar PowerShell o la CLI al implementar o administrar un sistema SAP NetWeaver en Azure.  

Como Azure ofrece más funciones, se agregarán nuevos cmdlets de PS que requieren una actualización de los cmdlets. Por lo tanto, es recomendable consultar el sitio de descargas de Azure al menos una vez al mes <https://azure.microsoft.com/downloads/> para ver si se ha publicado una nueva versión de los cmdlets. La nueva versión se instala sobre la anterior.

Para obtener una lista general de comandos de PowerShell relacionados con Azure, consulte aquí: <https://docs.microsoft.com/powershell/azure/>.

### <a name="management-via-microsoft-azure-cli-commands"></a>Administración a través de los comandos de CLI de Microsoft Azure

Los clientes que usan Linux y quieren administrar recursos de Azure pueden recurrir a PowerShell. Microsoft ofrece la CLI de Azure como una alternativa.
La CLI de Azure proporciona un conjunto de comandos de código abierto y multiplataforma para trabajar con la plataforma de Azure. La CLI de Azure proporciona muchas de las mismas funciones presentes en el Portal de Azure.

Para obtener más información sobre la instalación, la configuración y el uso de los comandos de la CLI para realizar tareas de Azure, consulte estos temas:

* [Install the Azure classic CLI][xplat-cli] (Instalación de la CLI de Azure clásica)
* [Implementación y administración de máquinas virtuales mediante plantillas de Azure Resource Manager y la CLI de Azure][../../linux/create-ssh-secured-vm-from-template.md]
* [Uso de la CLI de Azure para Mac, Linux y Windows con Azure Resource Manager][xplat-cli-azure-resource-manager]

Lea también el capítulo [CLI de Azure para máquinas virtuales Linux][deployment-guide-4.5.2] en la [Guía de implementación][planning-guide] a fin de descubrir cómo usar la CLI de Azure para implementar la extensión de Azure para SAP.


## <a name="first-steps-planning-a-deployment"></a>Primeros pasos del planeamiento de una implementación
El primer paso en el planeamiento de la implementación es NO comprobar las máquinas virtuales disponibles para ejecutar SAP. El primer paso puede requerir mucho tiempo, pero lo más importante es trabajar con los equipos de cumplimiento y seguridad de su empresa en cuáles son las condiciones de límite para implementar cada tipo de carga de trabajo de SAP o proceso de negocio en la nube pública. Si la empresa ha implementado anteriormente otro software en Azure, el proceso puede ser sencillo. Sin embargo, si la empresa nunca lo ha hecho, puede haber grandes debates para averiguar las condiciones del límite y las condiciones de seguridad que permiten hospedar determinados datos de SAP y procesos empresariales de SAP en la nube pública.

Puede consultar las [ofertas de cumplimiento de Microsoft](/microsoft-365/compliance/offering-home) para ver una lista de las ofertas de cumplimiento que Microsoft puede proporcionar. 

Otras áreas de preocupación como el cifrado de los datos en reposo u otro cifrado del servicio de Azure se documentan en [Introducción al cifrado de Azure](../../../security/fundamentals/encryption-overview.md).

No subestime esta fase del proyecto en el planeamiento. Solo cuando tenga un contrato y reglas en torno a este tema, debe ir al paso siguiente, que es el planeamiento de la arquitectura de red que implementa en Azure.


## <a name="different-ways-to-deploy-vms-for-sap-in-azure"></a>Diferentes formas de implementar máquinas virtuales para SAP en Azure

En este capítulo, se exponen las diferentes formas de implementar una máquina virtual en Azure. En este capítulo también se tratan los procedimientos de preparación adicionales, así como la administración de VHD y máquinas virtuales en Azure.

### <a name="deployment-of-vms-for-sap"></a>Implementación de máquinas virtuales para SAP

Microsoft Azure ofrece varias maneras de implementar máquinas virtuales y discos asociados. Por lo tanto, es importante reconocer las diferencias, puesto que los preparativos de las máquinas virtuales pueden variar según el método de implementación. En general, se analizan los siguientes escenarios:

#### <a name="moving-a-vm-from-on-premises-to-azure-with-a-non-generalized-disk"></a><a name="4d175f1b-7353-4137-9d2f-817683c26e53"></a>Procedimiento para mover máquinas virtuales del entorno local a Azure con un disco no generalizado

Planea mover un sistema SAP específico desde un entorno local a Azure. Esto puede hacerse cargando en Azure el VHD que contiene el sistema operativo, los archivos binarios de SAP y los de DBMS, así como los discos duros virtuales con los archivos de registro y de datos del DBMS. En contraposición al [segundo escenario siguiente][planning-guide-5.1.2], mantendrá el nombre de host, el SID de SAP y las cuentas de usuario de SAP en la máquina virtual de Azure, como se configuraron en el entorno local. Por lo tanto, no es necesario generalizar la imagen. Consulte el capítulo [Preparación para mover máquinas virtuales del entorno local a Azure con un disco no generalizado][planning-guide-5.2.1] de este documento para obtener información sobre los pasos de preparación del entorno local y la carga de máquinas virtuales o discos duros virtuales no generalizados en Azure. Lea el capítulo [Escenario 3: Mover una máquina virtual desde un entorno local con un discos duros virtuales no generalizado de Azure con SAP][deployment-guide-3.4] de la [Guía de implementación][deployment-guide] para conocer los pasos detallados para implementar dicha imagen en Azure.

#### <a name="deploying-a-vm-with-a-customer-specific-image"></a><a name="e18f7839-c0e2-4385-b1e6-4538453a285c"></a>Implementación de una máquina virtual con una imagen específica del cliente

Debido a los requisitos de revisión específicos de la versión del SO o DBMS, puede que las imágenes proporcionadas en Azure Marketplace no satisfagan sus necesidades. Por lo tanto, se recomienda crear una máquina virtual con su propia imagen privada de máquina virtual de SO o DBMS, que podrá implementarse varias veces más adelante. Para preparar una imagen privada con fines de duplicación, se deben tener en cuenta los siguientes aspectos:

---
> ![Windows][Logo_Windows] Windows
>
> Para obtener información, consulte: <https://docs.microsoft.com/azure/virtual-machines/windows/upload-generalized-managed> La configuración de Windows (como el nombre de host y el SID de Windows) debe abstraerse o generalizarse en la máquina virtual local mediante el comando sysprep.
>
>
> ![Linux][Logo_Linux] Linux
>
> Siga los pasos descritos en los siguientes artículos para [SUSE][virtual-machines-linux-create-upload-vhd-suse], [Red Hat][virtual-machines-linux-redhat-create-upload-vhd] u [Oracle Linux][virtual-machines-linux-create-upload-vhd-oracle], para preparar un disco duro virtual para cargarse en Azure.
>
>

---
Si ya ha instalado contenido de SAP en la máquina virtual local (especialmente en los sistemas de dos niveles), puede adaptar la configuración del sistema SAP después de realizar la implementación de la máquina virtual de Azure mediante el procedimiento de cambio de nombre de instancia admitido por el administrador de aprovisionamiento de software de SAP (nota de SAP [1619720]). Consulte los capítulos [Preparación para la implementación de una máquina virtual con una imagen específica del cliente para SAP][planning-guide-5.2.2] y [Carga de un VHD desde una instalación local a Azure][planning-guide-5.3.2] de este documento para obtener información sobre los pasos de preparación del entorno local y la carga de una máquina virtual generalizada en Azure. Lea el capítulo [Escenario 2: Implementación de una máquina virtual con una imagen personalizada para SAP][deployment-guide-3.3] de la [Guía de implementación][deployment-guide] para conocer los pasos detallados de la implementación de dicha imagen en Azure.

#### <a name="deploying-a-vm-out-of-the-azure-marketplace"></a>Implementación de máquinas virtuales de Azure Marketplace

Desea usar una imagen de máquina virtual proporcionada por Microsoft o un tercero en Azure Marketplace para implementar la máquina virtual. Una vez que se ha implementado la máquina virtual en Azure, sigue las mismas instrucciones y utilice las mismas herramientas para instalar el software de SAP o DBMS en la máquina virtual de la misma forma que lo haría en un entorno local. Para una descripción más detallada de la implementación, consulte el capítulo [Escenario 1: Implementación de una máquina virtual desde Azure Marketplace para SAP][deployment-guide-3.2] de la [Guía de implementación][deployment-guide].

### <a name="preparing-vms-with-sap-for-azure"></a><a name="6ffb9f41-a292-40bf-9e70-8204448559e7"></a>Preparación de máquinas virtuales con SAP para Azure

Antes de cargar las máquinas virtuales en Azure, debe asegurarse de que las máquinas virtuales y los discos duro virtuales cumplan determinados requisitos. Existen pequeñas diferencias según el método de implementación que se utilice.

#### <a name="preparation-for-moving-a-vm-from-on-premises-to-azure-with-a-non-generalized-disk"></a><a name="1b287330-944b-495d-9ea7-94b83aff73ef"></a>Preparación para mover máquinas virtuales del entorno local a Azure con un disco no generalizado

Un método de implementación habitual consiste en mover una máquina virtual existente en la que se ejecuta un sistema SAP del entorno local a Azure. Esa máquina virtual y el sistema SAP incluido en ella deben ejecutarse en Azure con el mismo nombre de host y, con toda probabilidad, el mismo SID de SAP. En este caso, el sistema operativo invitado de la máquina virtual no debe generalizarse para varias implementaciones. Si la red local se extendió a Azure, incluso se pueden usar las mismas cuentas de dominio dentro de la máquina virtual, ya que se usaron antes en el entorno local.

Estos son los requisitos que se deben cumplir a la hora de preparar su propio disco de VM de Azure:

* Originalmente, el VHD que contiene el sistema operativo podía tener un tamaño máximo de 127 GB. Esta limitación se eliminó a finales de marzo de 2015. Ahora, el VHD que contiene el sistema operativo puede tener un tamaño máximo de 1 TB, al igual que cualquier otro VHD hospedado en Azure Storage.
* Debe tener el formato de VHD fijo. Los VHD dinámicos o en formato VHDx aún no se admiten en Azure. Los VHD dinámicos se convertirán en VHD estáticos al cargarlos con los cmdlets de PowerShell o la CLI.
* Los discos duros virtuales que estén montados en la máquina virtual y se deban volver a montar en ella en Azure tienen que tener también un formato de disco duro virtual fijo. Lea [este artículo (Linux)](../../linux/managed-disks-overview.md) y [este otro (Windows)](../../windows/managed-disks-overview.md) para conocer los límites de tamaño de los discos de datos. Los VHD dinámicos se convertirán en VHD estáticos al cargarlos con los cmdlets de PowerShell o la CLI.
* Agregue otra cuenta local con privilegios de administrador que puede utilizar el servicio de soporte técnico de Microsoft, o que se puede asignar como contexto para que se ejecuten los servicios y las aplicaciones hasta que se implemente la máquina virtual y se puedan utilizar usuarios más adecuados.
* Agregue otras cuentas locales, ya que se podrían necesitar para el escenario de implementación concreto.

---
> ![Windows][Logo_Windows] Windows
>
> En este escenario, no hace falta generalizar (Sysprep) la máquina virtual para cargarla e implementarla en Azure.
> Asegúrese de que la unidad D:\ no se esté usando.
> Configure el montaje automático de los discos conectados tal y como se describe en el capítulo [Configuración de montaje automático para discos conectados][planning-guide-5.5.3] del presente documento.
>
> ![Linux][Logo_Linux] Linux
>
> En este escenario, no hace falta generalizar (waagent -deprovision) la máquina virtual para cargarla e implementarla en Azure.
> Asegúrese de que no se usa /mnt/resource y de que todos los discos se montan mediante UUID. En lo que respecta al disco del SO, asegúrese de que la entrada del cargador de arranque también refleja el montaje basado en UUID.
>
>

---
#### <a name="preparation-for-deploying-a-vm-with-a-customer-specific-image-for-sap"></a><a name="57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3"></a>Preparación para la implementación de una máquina virtual con una imagen específica del cliente para SAP

Los archivos VHD que contienen un SO generalizado se almacenan en contenedores en cuentas de Azure Storage o como imágenes de Managed Disks. Puede implementar una nueva máquina virtual de dicha imagen si hace referencia al VHD o a la imagen de Managed Disks como un origen en los archivos de plantilla de implementación, tal y como se describe en el capítulo [Escenario 2: Implementación de una máquina virtual con una imagen personalizada para SAP][deployment-guide-3.3] de la [Guía de implementación][deployment-guide].

Estos son los requisitos que se deben cumplir a la hora de preparar su propia imagen de VM de Azure:

* Originalmente, el VHD que contiene el sistema operativo podía tener un tamaño máximo de 127 GB. Esta limitación se eliminó a finales de marzo de 2015. Ahora, el VHD que contiene el sistema operativo puede tener un tamaño máximo de 1 TB, al igual que cualquier otro VHD hospedado en Azure Storage.
* Debe tener el formato de VHD fijo. Los VHD dinámicos o en formato VHDx aún no se admiten en Azure. Los VHD dinámicos se convertirán en VHD estáticos al cargarlos con los cmdlets de PowerShell o la CLI.
* Los discos duros virtuales que estén montados en la máquina virtual y se deban volver a montar en ella en Azure tienen que tener también un formato de disco duro virtual fijo. Lea [este artículo (Linux)](../../windows/managed-disks-overview.md) y [este otro (Windows)](../../linux/managed-disks-overview.md) para conocer los límites de tamaño de los discos de datos. Los VHD dinámicos se convertirán en VHD estáticos al cargarlos con los cmdlets de PowerShell o la CLI.
* Agregue otras cuentas locales, ya que se podrían necesitar para el escenario de implementación concreto.
* Si la imagen contiene una instalación de SAP NetWeaver y es probable que se vaya a modificar el nombre original del host en el momento de la implementación de Azure, se recomienda copiar en la plantilla las últimas versiones del DVD del administrador de aprovisionamiento de software de SAP. Así podrá utilizar fácilmente la función de cambio de nombre de SAP proporciona para adaptar el nombre de host cambiado o modificar al SID del sistema SAP dentro de la imagen de la máquina virtual implementada en cuanto se inicie una nueva copia.

---
> ![Windows][Logo_Windows] Windows
>
> Asegúrese de que la unidad D:\ no se esté usando. Configure el montaje automático para los discos conectados tal y como se describe en el capítulo [Configuración de montaje automático para discos conectados][planning-guide-5.5.3] del presente documento.
>
> ![Linux][Logo_Linux] Linux
>
> Asegúrese de que no se usa /mnt/resource y de que todos los discos se montan mediante UUID. En lo que respecta al disco del SO, cerciórese de que la entrada del cargador de arranque también refleja el montaje basado en UUID.
>
>

---
* GUI de SAP (para fines de administración e instalación) pueden estar ya instalados en dicha plantilla.
* Es posible instalar cualquier otro software necesario para ejecutar la máquina virtual correctamente en escenarios entre locales siempre que este pueda funcionar con el cambio de nombre de la VM.

Si la máquina virtual está lo suficientemente preparada para ser genérica y finalmente independiente de cuentas o usuarios no disponibles en el escenario de implementación de Azure objetivo, se lleva a cabo el último paso de preparación, que consiste en la generalización de la imagen.

##### <a name="generalizing-a-vm"></a>Generalización de una máquina virtual
---
> ![Windows][Logo_Windows] Windows
>
> El último paso es iniciar sesión en una máquina virtual con una cuenta de administrador. Abra una ventana de comandos de Windows como *administrador*. Vaya a %windir%\windows\system32\sysprep y ejecute sysprep.exe.
> Se mostrará una pequeña ventana. Es importante marcar la opción **Generalizar** (estará desmarcada de manera predeterminada) y cambiar la opción de apagado de "Reiniciar" (el valor predeterminado) a "Apagar". Para este procedimiento, se supone que el proceso de sysprep se ejecuta localmente en el SO invitado de una máquina virtual.
> Si desea llevar a cabo el procedimiento con una máquina virtual que ya se ejecute en Azure, siga los pasos que se describen en [este artículo](../../windows/capture-image-resource.md).
>
> ![Linux][Logo_Linux] Linux
>
> [Cómo capturar una máquina virtual Linux para usarla como plantilla de Resource Manager][capture-image-linux-step-2-create-vm-image]
>
>

---
### <a name="transferring-vms-and-vhds-between-on-premises-to-azure"></a>Transferencia de máquinas y discos duros virtuales entre locales a Azure
Como no es posible cargar discos e imágenes de máquinas virtuales en Azure mediante Azure Portal, tendrá que usar los cmdlets de Azure PowerShell o la CLI. Otra posibilidad es utilizar la herramienta "AzCopy". Esta herramienta puede copiar discos duros virtuales entre locales y Azure (en ambas direcciones). También puede copiar discos duros virtuales entre las regiones de Azure. Consulte [esta documentación][storage-use-azcopy] para la descarga y el uso de AzCopy.

Una tercera alternativa sería usar varias herramientas orientadas a GUI de terceros. De todas formas, asegúrese de que estas herramientas son compatibles con los blobs en páginas de Azure. Para nuestros propósitos, es necesario usar el almacén de blobs en páginas de Azure (las diferencias se describen aquí: <https://docs.microsoft.com/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs>). Por su parte, las herramientas que proporciona Azure también resultan muy eficaces a la hora de comprimir las máquinas y los discos duros virtuales que se tienen que cargar. Se trata de un factor importante, ya que esta eficacia en la compresión permite reducir el tiempo de carga (que varía según el vínculo de carga a Internet de las instalaciones locales y la región de implementación de Azure objetivo). En este sentido, resulta razonable suponer que la carga de una máquina virtual o un disco duro virtual desde una ubicación europea en los centros de datos de Azure situados en los EE. UU. tardará más que si se cargan los mismos elementos a los centros de datos europeos de Azure.

#### <a name="uploading-a-vhd-from-on-premises-to-azure"></a><a name="a43e40e6-1acc-4633-9816-8f095d5a7b6a"></a>Carga de un VHD desde una instalación local a Azure
Para cargar una máquina virtual o un disco duro virtual existentes desde la red local, estos deben cumplir los requisitos especificados en el capítulo [Preparación para mover máquinas virtuales del entorno local a Azure con un disco no generalizado][planning-guide-5.2.1] de este documento.

No hace falta generalizar dicha máquina virtual, sino que se puede cargar con el estado y la forma que tenga tras su apagado en el lado local. Lo mismo se aplica a los discos duros virtuales adicionales que no contengan ningún sistema operativo.

##### <a name="uploading-a-vhd-and-making-it-an-azure-disk"></a>Carga de un VHD y su conversión en un disco de Azure
En este caso, queremos cargar un VHD (con o sin un sistema operativo en él) y montarlo en una máquina virtual como un disco de datos o utilizarlo a modo de disco del SO. Se trata de un proceso compuesto por varios pasos.

**PowerShell**

* Inicie sesión en su suscripción con *Connect-AzAccount*.
* Establezca la suscripción de su contexto con *Set-AzContext* y el parámetro SubscriptionId o SubscriptionName (consulte <https://docs.microsoft.com/powershell/module/az.accounts/set-Azcontext>).
* Cargue el disco duro virtual con *Add-AzVhd* en una cuenta de Azure Storage (consulte <https://docs.microsoft.com/powershell/module/az.compute/add-Azvhd>).
* (Opcional) Cree un disco administrado desde el disco duro virtual con *New-AzDisk* (consulte <https://docs.microsoft.com/powershell/module/az.compute/new-Azdisk>).
* Establezca el disco del sistema operativo de una nueva configuración de máquina virtual como disco duro virtual o disco administrado con *Set-AzVMOSDisk* (consulte <https://docs.microsoft.com/powershell/module/az.compute/set-Azvmosdisk>).
* Cree una nueva máquina virtual desde la configuración de máquina virtual con *New-AzVM* (consulte <https://docs.microsoft.com/powershell/module/az.compute/new-Azvm>).
* Agregue un disco de datos a una nueva máquina virtual con *Add-AzVMDataDisk* (consulte <https://docs.microsoft.com/powershell/module/az.compute/add-Azvmdatadisk>).

**CLI de Azure**

* Inicie sesión en la suscripción con *az login*
* Seleccione su suscripción con *az account set --subscription `<subscription name or id`>* .
* Cargue el disco duro virtual con *az storage blob upload* (consulte [Using the Azure CLI with Azure Storage][storage-azure-cli] [Uso de la CLI de Azure con Azure Storage]).
* (Opcional) Cree un disco administrado desde el disco duro virtual con *az disk create* (consulte https://docs.microsoft.com/cli/azure/disk ).
* Cree una máquina virtual especificando el VHD o el disco de Managed Disks cargados como disco del sistema operativo con *az vm create* y el parámetro *--attach-os-disk*.
* Agregue un disco de datos a una nueva máquina virtual con *az vm disk attach* y el parámetro *--new*.

**Plantilla**

* Cargue el VHD con PowerShell o la CLI de Azure.
* (Opcional) Cree un disco administrado a partir del disco duro virtual con PowerShell, la CLI de Azure o Azure Portal.
* Implemente la VM con una plantilla de JSON en la que se haga referencia al VHD, como se muestra en [esta plantilla de JSON de ejemplo](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-specialized-vhd-new-or-existing-vnet/azuredeploy.json) o con un disco de Managed Disks, como se muestra en [esta plantilla de JSON de ejemplo](https://github.com/Azure/azure-quickstart-templates/blob/master/sap-2-tier-user-image-md/azuredeploy.json).

#### <a name="deployment-of-a-vm-image"></a>Implementación de una imagen de máquina virtual
Para cargar una máquina virtual o un disco duro virtual existentes desde la red local a fin de poder utilizarlos como imagen de máquina virtual de Azure, estos deberán cumplir los requisitos enumerados en el capítulo [Preparación para la implementación de una máquina virtual con una imagen específica del cliente para SAP][planning-guide-5.2.2] de este documento.

* Utilice *sysprep* en Windows o *waagent -deprovision* en Linux para generalizar la máquina virtual (consulte [Sysprep Technical Reference](/previous-versions/windows/it-pro/windows-vista/cc766049(v=ws.10)) [Referencia técnica de Sysprep] para Windows o el artículo de [Captura de una máquina virtual Linux para usarla como plantilla de Resource Manager][capture-image-linux-step-2-create-vm-image] para Linux).
* Inicie sesión en su suscripción con *Connect-AzAccount*.
* Establezca la suscripción de su contexto con *Set-AzContext* y el parámetro SubscriptionId o SubscriptionName (consulte <https://docs.microsoft.com/powershell/module/az.accounts/set-Azcontext>).
* Cargue el disco duro virtual con *Add-AzVhd* en una cuenta de Azure Storage (consulte <https://docs.microsoft.com/powershell/module/az.compute/add-Azvhd>).
* (Opcional) Cree una imagen de disco administrado desde el disco duro virtual con *New-AzImage* (consulte <https://docs.microsoft.com/powershell/module/az.compute/new-Azimage>).
* Configure el disco del sistema operativo de una nueva configuración de máquina virtual para:
  * Disco duro virtual con *Set-AzVMOSDisk -SourceImageUri -CreateOption fromImage* (consulte <https://docs.microsoft.com/powershell/module/az.compute/set-Azvmosdisk>).
  * Imagen de disco administrado *Set-AzVMSourceImage* (consulte <https://docs.microsoft.com/powershell/module/az.compute/set-Azvmsourceimage>).
* Cree una nueva máquina virtual desde la configuración de máquina virtual con *New-AzVM* (consulte <https://docs.microsoft.com/powershell/module/az.compute/new-Azvm>).

**CLI de Azure**

* Utilice *sysprep* en Windows o *waagent -deprovision* en Linux para generalizar la máquina virtual (consulte [Sysprep Technical Reference](/previous-versions/windows/it-pro/windows-vista/cc766049(v=ws.10)) [Referencia técnica de Sysprep] para Windows o el artículo de [Captura de una máquina virtual Linux para usarla como plantilla de Resource Manager][capture-image-linux-step-2-create-vm-image] para Linux).
* Inicie sesión en la suscripción con *az login*
* Seleccione su suscripción con *az account set --subscription `<subscription name or id`>* .
* Cargue el disco duro virtual con *az storage blob upload* (consulte [Using the Azure CLI with Azure Storage][storage-azure-cli] [Uso de la CLI de Azure con Azure Storage]).
* (Opcional) Cree una imagen de disco administrado desde el disco duro virtual con *az image create* (consulte https://docs.microsoft.com/cli/azure/image ).
* Cree una máquina virtual especificando el VHD o la imagen de Managed Disks cargados como disco del sistema operativo con *az vm create* y el parámetro *--image*.

**Plantilla**

* Utilice *sysprep* en Windows o *waagent -deprovision* en Linux para generalizar la máquina virtual (consulte [Sysprep Technical Reference](/previous-versions/windows/it-pro/windows-vista/cc766049(v=ws.10)) [Referencia técnica de Sysprep] para Windows o el artículo de [Captura de una máquina virtual Linux para usarla como plantilla de Resource Manager][capture-image-linux-step-2-create-vm-image] para Linux).
* Cargue el VHD con PowerShell o la CLI de Azure.
* (Opcional) Cree una imagen de los discos administrados a partir del disco duro virtual con PowerShell, la CLI de Azure o Azure Portal.
* Implemente la VM con una plantilla de JSON en la que se haga referencia al VHD de la imagen, como se muestra en [esta plantilla de JSON de ejemplo](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-specialized-vhd-new-or-existing-vnet/azuredeploy.json) o con una imagen de Managed Disks, como se muestra en [esta plantilla de JSON de ejemplo](https://github.com/Azure/azure-quickstart-templates/blob/master/sap-2-tier-user-image-md/azuredeploy.json).

#### <a name="downloading-vhds-or-managed-disks-to-on-premises"></a>Descarga local de discos duros virtuales o discos de Managed Disks
La solución de infraestructura como servicio de Azure no solo funciona unidireccionalmente, en el sentido de que solo permite cargar sistemas SAP y VHD, sino que también puede mover sistemas SAP de Azure de nuevo al entorno local.

Durante la descarga, los discos duros virtuales o los discos de Managed Disks no pueden estar activos. Incluso cuando se descargan los discos que están montados en máquinas virtuales, la máquina virtual tiene que estar apagada y desasignada. Si solo quiere descargar el contenido de la base de datos, que después se debe usar para configurar un nuevo sistema local, y si desea que, durante el tiempo que dure la descarga y la configuración del nuevo sistema, el sistema presente en Azure continúe estando operativo, podría evitar un tiempo de inactividad prolongado realizando una copia de seguridad comprimida de la base de datos en un disco para sencillamente descargar este último en lugar de descargar también la máquina virtual base del sistema operativo.

#### <a name="powershell"></a>PowerShell

* Descarga de un disco de Managed Disks  
  En primer lugar, debe acceder al blob subyacente del disco de Managed Disks. Después, puede copiar el blob subyacente en una cuenta de almacenamiento nueva y descargarlo desde esta cuenta.

  ```powershell
  $access = Grant-AzDiskAccess -ResourceGroupName <resource group> -DiskName <disk name> -Access Read -DurationInSecond 3600
  $key = (Get-AzStorageAccountKey -ResourceGroupName <resource group> -Name <storage account name>)[0].Value
  $destContext = (New-AzStorageContext -StorageAccountName <storage account name -StorageAccountKey $key)
  Start-AzStorageBlobCopy -AbsoluteUri $access.AccessSAS -DestContainer <container name> -DestBlob <blob name> -DestContext $destContext
  # Wait for blob copy to finish
  Get-AzStorageBlobCopyState -Container <container name> -Blob <blob name> -Context $destContext
  Save-AzVhd -SourceUri <blob in new storage account> -LocalFilePath <local file path> -StorageKey $key
  # Wait for download to finish
  Revoke-AzDiskAccess -ResourceGroupName <resource group> -DiskName <disk name>
  ```

* Descarga de un disco duro virtual  
  Una vez que se haya detenido el sistema SAP y se haya apagado la VM, podrá usar el cmdlet de PowerShell `Save-AzVhd` en el destino local para descargar los discos VHD en el entorno local. Para ello, necesita la dirección URL del disco duro virtual, que puede encontrar en la sección "Storage" de Azure Portal (tiene que ir a la cuenta de almacenamiento y al contenedor de almacenamiento donde se creó el disco duro virtual), y tiene que saber dónde se debe copiar el disco duro virtual.

  Entonces podrá aprovechar el comando con solo definir el parámetro SourceUri como la URL del disco duro virtual que se va a descargar y LocalFilePath como la ubicación física del disco duro virtual (incluido su nombre). El comando podría presentar una sintaxis parecida a esta:

  ```powerhell
  Save-AzVhd -ResourceGroupName <resource group name of storage account> -SourceUri http://<storage account name>.blob.core.windows.net/<container name>/sapidedata.vhd -LocalFilePath E:\Azure_downloads\sapidesdata.vhd
  ```

  Para más información acerca del cmdlet Save-AzVhd, consulte aquí <https://docs.microsoft.com/powershell/module/az.compute/save-Azvhd>.

#### <a name="azure-cli"></a>Azure CLI
* Descarga de un disco de Managed Disks  
  En primer lugar, debe acceder al blob subyacente del disco de Managed Disks. Después, puede copiar el blob subyacente en una cuenta de almacenamiento nueva y descargarlo desde esta cuenta.

  ```azurecli
  az disk grant-access --ids "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>" --duration-in-seconds 3600
  az storage blob download --sas-token "<sas token>" --account-name <account name> --container-name <container name> --name <blob name> --file <local file>
  az disk revoke-access --ids "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>"
  ```

* Descarga de un disco duro virtual   
  Una vez que se haya detenido el sistema SAP y se haya apagado la VM, podrá usar el comando de la CLI de Azure `_azure storage blob download_` en el destino local para volver a descargar los discos VHD al entorno local. Para ello, necesita el nombre y el contenedor del disco duro virtual, que puede encontrar en la sección "Storage" de Azure Portal (tiene que ir a la cuenta de almacenamiento y al contenedor de almacenamiento donde se creó el disco duro virtual), y tiene que saber dónde se debe copiar el disco duro virtual.

  Entonces podrá aprovechar el comando definiendo los parámetros blob y container del disco duro virtual que se va a descargar, y destination como la ubicación física de destino del disco duro virtual (incluido su nombre). El comando podría presentar una sintaxis parecida a esta:

  ```azurecli
  az storage blob download --name <name of the VHD to download> --container-name <container of the VHD to download> --account-name <storage account name of the VHD to download> --account-key <storage account key> --file <destination of the VHD to download>
  ```

### <a name="transferring-vms-and-disks-within-azure"></a>Transferencia de máquinas virtuales y discos dentro de Azure

#### <a name="copying-sap-systems-within-azure"></a>Copia de los sistemas SAP en Azure

Es probable que un sistema SAP o incluso un servidor de DBMS dedicado que respalde una capa de la aplicación de SAP se componga de varios discos que contengan el sistema operativo con los archivos binarios o los archivos de registro y datos de la base de datos de SAP. Ni la funcionalidad de copiar discos de Azure ni la de guardar dichos discos en un disco local cuenta con un mecanismo de sincronización que pueda realizar instantáneas de varios discos de una forma consistente. Por lo tanto, el estado de los discos copiados o guardados, aunque estén montados en la misma máquina virtual, sería distinto. Es decir, en el caso concreto de tener distintos archivos de registro y datos incluidos en los distintos discos, la base de datos terminaría siendo incoherente.

**Conclusión: para copiar o guardar discos que formen parte de la configuración de un sistema SAP, tendrá que detener dicho sistema y también apagar las máquinas virtuales implementadas. Solo entonces podrá copiar o descargar el conjunto de discos para crear una copia del sistema SAP en Azure o en el entorno local.**

Los discos de datos pueden almacenarse como archivos VHD en una cuenta de Azure Storage y se pueden asociar a una máquina virtual directamente o utilizarse como una imagen. En este caso, el VHD se copia en otra ubicación antes de asociarse a la máquina virtual. El nombre completo del archivo VHD de Azure debe ser único dentro de Azure. Como ya se ha mencionado anteriormente, el nombre se compone de tres partes y presenta una sintaxis similar a esta:

`http(s)://<storage account name>.blob.core.windows.net/<container name>/<vhd name>`

Los discos de datos también pueden ser discos de Managed Disks. En este caso, el disco de Managed Disks se usa para crear un disco de Managed Disks antes de conectarlo a la máquina virtual. El nombre del disco de Managed Disks debe ser exclusivo dentro de un grupo de recursos.

##### <a name="powershell"></a>PowerShell

Puede usar cmdlets de Azure PowerShell para copiar un disco duro virtual, como se muestra en [este artículo][storage-powershell-guide-full-copy-vhd]. Para crear un disco de Managed Disks, use New-AzDiskConfig y New-AzDisk como se muestra en el ejemplo siguiente.

```powershell
$config = New-AzDiskConfig -CreateOption Copy -SourceUri "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>" -Location <location>
New-AzDisk -ResourceGroupName <resource group name> -DiskName <disk name> -Disk $config
```

##### <a name="azure-cli"></a>Azure CLI

Puede usar la CLI de Azure para copiar un VHD. Para crear un disco de Managed Disks, use *az disk create* como se muestra en el ejemplo siguiente.

```azurecli
az disk create --source "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>" --name <disk name> --resource-group <resource group name> --location <location>
```

##### <a name="azure-storage-tools"></a>Herramientas de Azure Storage

* <https://storageexplorer.com/>

Las ediciones profesionales de los Exploradores de Azure Storage se pueden encontrar aquí:

* <https://www.cerebrata.com/>
* <https://clumsyleaf.com/products/cloudxplorer>

La copia de un disco duro virtual dentro de una cuenta de almacenamiento es un proceso que solo tarda unos pocos segundos (como sucede en el caso de la creación de instantáneas con copia diferida y copia en escritura por parte de hardware de SAN). Una vez que tenga una copia del archivo de disco duro virtual, podrá asociarlo a una máquina virtual o usarlo como una imagen para conectar copias del disco duro virtual a máquinas virtuales.

##### <a name="powershell"></a>PowerShell

```powershell
# attach a vhd to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$vm = Add-AzVMDataDisk -VM $vm -Name newdatadisk -VhdUri <path to vhd> -Caching <caching option> -DiskSizeInGB $null -Lun <lun, for example 0> -CreateOption attach
$vm | Update-AzVM

# attach a managed disk to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$vm = Add-AzVMDataDisk -VM $vm -Name newdatadisk -ManagedDiskId <managed disk id> -Caching <caching option> -DiskSizeInGB $null -Lun <lun, for example 0> -CreateOption attach
$vm | Update-AzVM

# attach a copy of the vhd to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$vm = Add-AzVMDataDisk -VM $vm -Name <disk name> -VhdUri <new path of vhd> -SourceImageUri <path to image vhd> -Caching <caching option> -DiskSizeInGB $null -Lun <lun, for example 0> -CreateOption fromImage
$vm | Update-AzVM

# attach a copy of the managed disk to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$diskConfig = New-AzDiskConfig -Location $vm.Location -CreateOption Copy -SourceUri <source managed disk id>
$disk = New-AzDisk -DiskName <disk name> -Disk $diskConfig -ResourceGroupName <resource group name>
$vm = Add-AzVMDataDisk -VM $vm -Caching <caching option> -Lun <lun, for example 0> -CreateOption attach -ManagedDiskId $disk.Id
$vm | Update-AzVM
```

##### <a name="azure-cli"></a>Azure CLI

```azurecli

# attach a vhd to a vm
az vm unmanaged-disk attach --resource-group <resource group name> --vm-name <vm name> --vhd-uri <path to vhd>

# attach a managed disk to a vm
az vm disk attach --resource-group <resource group name> --vm-name <vm name> --disk <managed disk id>

# attach a copy of the vhd to a vm
# this scenario is currently not possible with Azure CLI. A workaround is to manually copy the vhd to the destination.

# attach a copy of a managed disk to a vm
az disk create --name <new disk name> --resource-group <resource group name> --location <location of target virtual machine> --source <source managed disk id>
az vm disk attach --disk <new disk name or managed disk id> --resource-group <resource group name> --vm-name <vm name> --caching <caching option> --lun <lun, for example 0>
```

#### <a name="copying-disks-between-azure-storage-accounts"></a><a name="9789b076-2011-4afa-b2fe-b07a8aba58a1"></a>Copia de discos entre cuentas de Azure Storage
Esta tarea no puede realizarse en Azure Portal. Puede usar cmdlets de Azure PowerShell, la CLI de Azure o un explorador de almacenamiento de terceros. Los cmdlets de PowerShell o los comandos de la CLI pueden crear y administrar blobs, lo que abarca la capacidad de copiar blobs de forma asincrónica entre las distintas cuentas de almacenamiento y regiones de la suscripción de Azure.

##### <a name="powershell"></a>PowerShell
También puede copiar discos duros virtuales entre suscripciones. Para más información, consulte [este artículo][storage-powershell-guide-full-copy-vhd].

El flujo básico de la lógica de los cmdlets de PowerShell se compone, a grandes rasgos, de estos pasos:

* Crear un contexto de cuenta de almacenamiento para la cuenta de almacenamiento de **origen** con *New-AzStorageContext* (consulte <https://docs.microsoft.com/powershell/module/az.storage/new-AzStoragecontext>)
* Crear un contexto de cuenta de almacenamiento para la cuenta de almacenamiento de **destino** con *New-AzStorageContext* (consulte <https://docs.microsoft.com/powershell/module/az.storage/new-AzStoragecontext>)
* Inicie la copia con

```powershell
Start-AzStorageBlobCopy -SrcBlob <source blob name> -SrcContainer <source container name> -SrcContext <variable containing context of source storage account> -DestBlob <target blob name> -DestContainer <target container name> -DestContext <variable containing context of target storage account>
```

* Compruebe el estado de la copia en un bucle con

```powershell
Get-AzStorageBlobCopyState -Blob <target blob name> -Container <target container name> -Context <variable containing context of target storage account>
```

* Asocie el nuevo VHD a una máquina virtual como se describió anteriormente.

Consulte [este artículo][storage-powershell-guide-full-copy-vhd] para ejemplos.

##### <a name="azure-cli"></a>Azure CLI
* Inicie la copia con

```azurecli
az storage blob copy start --source-blob <source blob name> --source-container <source container name> --source-account-name <source storage account name> --source-account-key <source storage account key> --destination-container <target container name> --destination-blob <target blob name> --account-name <target storage account name> --account-key <target storage account name>
```

* Compruebe el estado si la copia está en un bucle con

```azurecli
az storage blob show --name <target blob name> --container <target container name> --account-name <target storage account name> --account-key <target storage account name>
```

* Asocie el nuevo VHD a una máquina virtual como se describió anteriormente.

### <a name="disk-handling"></a>Administración de discos

#### <a name="vmdisk-structure-for-sap-deployments"></a><a name="4efec401-91e0-40c0-8e64-f2dceadff646"></a>Estructura de VM/disco para implementaciones de SAP

Lo ideal es que la administración de la estructura de una máquina virtual y los discos asociados sea muy sencilla. En las instalaciones locales, los clientes han desarrollado muchas formas de estructurar una instalación de servidor.

* Un disco base que contenga el sistema operativo y todos los archivos binarios de DBMS o SAP. Desde marzo de 2015, este disco puede tener un tamaño máximo de 1 TB, en lugar de las restricciones anteriores que lo limitaban a 127 GB.
* Uno o varios discos que contengan el archivo de registro de DBMS de la base de datos de SAP y el del área de almacenamiento temporal de DBMS (si el DBMS es compatible con esto). Si los requisitos de E/S por segundo de registro de la base de datos son elevados, tendrá que seccionar varios discos para lograr el volumen de E/S por segundo necesario.
* Diversos discos que contengan uno o dos archivos de base de datos de la base de datos de SAP, así como los archivos de datos temporales de DBMS (si el DBMS es compatible con esto).

![Configuración de referencia de máquinas virtuales IaaS de Azure en SAP][planning-guide-figure-1300]


---
> ![Windows][Logo_Windows] Windows
>
> En el caso de muchos clientes, observamos configuraciones en las que, por ejemplo, los archivos binarios de SAP y DBMS no estaban instalados en la unidad c:\ en la que sí lo estaba el sistema operativo. Existían varios motivos para ello, pero cuando nos fijábamos en la causa principal, normalmente se debía a que los discos eran pequeños y actualizaciones del sistema operativo necesitaban más espacio hace 10 o 15 años. Ambas condiciones han dejado de aplicarse actualmente. En la actualidad, la unidad c:\ puede asignarse a máquinas virtuales o discos de gran volumen. A fin de simplificar la estructura de las implementaciones, se recomienda seguir el patrón de implementación para sistemas SAP NetWeaver en Azure que verá a continuación.
>
> El archivo de paginación del sistema operativo de Windows debe estar en la unidad D:\ (disco no persistente).
>
> ![Linux][Logo_Linux] Linux
>
> Coloque el archivo de intercambio de Linux en /mnt/mnt/resource en Linux, tal y como se describe en [este artículo][virtual-machines-linux-agent-user-guide]. El archivo de intercambio puede configurarse en el archivo de configuración del /etc/waagent.conf del agente de Linux. Agregue o cambie las siguientes opciones de configuración:
>
>

```console
ResourceDisk.EnableSwap=y
ResourceDisk.SwapSizeMB=30720
```

Para activar los cambios, debe reiniciar al agente de Linux con

```console
sudo service waagent restart
```

Lea la nota de SAP [1597355] para más información sobre el tamaño recomendado del archivo de intercambio.

---
Los requisitos de E/S por segundo y la latencia necesaria determinan el número de discos utilizados para los archivos de datos de DBMS y el tipo de almacenamiento de Azure Storage en el que se hospedan estos discos. Las cuotas exactas se describen en [este artículo (Linux)][virtual-machines-sizes-linux] y en [este otro (Windows)][virtual-machines-sizes-windows].

La experiencia en implementaciones de SAP que hemos acumulado durante los últimos 2 años nos ha permitido aprender una serie de lecciones que se pueden resumir del siguiente modo:

* El tráfico de IOPS a los diferentes archivos de datos no es siempre el mismo porque puede que los sistemas existentes del cliente presenten archivos de datos de distinto tamaño, los cuales que representan sus bases de datos de SAP. Por tanto, resultó ser más recomendable usar una configuración de RAID en lugar de varios discos para colocar los LUN de archivos de datos extraídos de estos. Se han producido situaciones, especialmente con Azure Standard Storage, en las que una tasa E/S por segundo alcanzó la cuota de un solo disco en relación con el registro de transacciones del DBMS. En estos casos, se recomienda usar Premium Storage o, como método alternativo, agregar varios discos de Standard Storage con una sección de software.

---
> ![Windows][Logo_Windows] Windows
>
> * [Prácticas recomendadas para mejorar el rendimiento para SQL Server en Azure Virtual Machines][virtual-machines-sql-server-performance-best-practices]
>
> ![Linux][Logo_Linux] Linux
>
> * [Configuración del software RAID en Linux][virtual-machines-linux-configure-raid]
> * [Configuración del LVM en una máquina virtual Linux en Azure][virtual-machines-linux-configure-lvm]
>
>

---
* Premium Storage muestra un rendimiento bastante superior, especialmente en el caso de escrituras del registro de transacciones cruciales. Para escenarios de SAP en los que se espera obtener un rendimiento similar al de un entorno de producción, se recomienda encarecidamente usar una serie de máquinas virtuales que puedan aprovechar Azure Premium Storage.

Tenga en cuenta que el disco, que contiene el sistema operativo, y, como se recomienda, también los archivos binarios de SAP y la base de datos (VM base), ya no está limitado a 127 GB. Ahora puede tener un tamaño máximo de 1 TB. Con este límite, se contará con espacio suficiente para guardar todos los archivos necesarios, incluidos los registros de trabajos por lotes de SAP, por ejemplo.

Para más sugerencias e información, en concreto para las máquinas virtuales de DBMS, consulte la [Guía de implementación de DBMS][dbms-guide]

#### <a name="disk-handling"></a>Administración de discos

En la mayoría de los escenarios, tendrá que crear discos adicionales para implementar la base de datos de SAP en la máquina virtual. En el capítulo [Estructura de VM/disco para implementaciones de SAP][planning-guide-5.5.1] de este documento ya se trataron las consideraciones sobre el número de discos. Azure Portal permite asociar y desasociar discos una vez implementada una máquina virtual base. Los discos se pueden asociar y desasociar cuando la máquina virtual se está ejecutando, así como cuando se encuentra detenida. Al asociar un disco, Azure Portal ofrece la posibilidad de asociar un disco vacío o uno existente que en ese momento no esté conectado a ninguna otra máquina virtual.

**Nota**: Los discos solo pueden estar asociados a una máquina virtual al mismo tiempo.

![Conexión/desconexión de discos con Azure Standard Storage][planning-guide-figure-1400]

Durante la implementación de una nueva máquina virtual, puede decidir si desea usar discos de Managed Disks o colocar los discos en las cuentas de Azure Storage. Si desea usar Premium Storage, se recomienda utilizar discos de Managed Disks.

Después, debe decidir si desea crear un disco vacío o si quiere seleccionar un disco existente que se haya cargado con anterioridad y deba asociar ahora a la VM.

**IMPORTANTE**: **NO** se recomienda usar el almacenamiento en caché de host con Azure Standard Storage. Debe dejar la preferencia de caché de host en el valor predeterminado, es decir, NINGUNO. Con Azure Premium Storage, debería habilitar el almacenamiento en caché de lectura si la característica de E/S es principalmente de lectura, como es típico del tráfico de E/S relativo a archivos de datos de bases de datos. En el caso del archivo de registro de transacciones, se recomienda no utilizar el almacenamiento en caché.

---
> ![Windows][Logo_Windows] Windows
>
> [Conexión de un disco de datos en Azure Portal][virtual-machines-linux-attach-disk-portal]
>
> Si los discos se han adjuntado, tiene que iniciar sesión en la máquina virtual para abrir el administrador de discos de Windows. Si el montaje automático no está habilitado como se recomienda en el capítulo [Configuración de montaje automático para discos conectados][planning-guide-5.5.3], el volumen recién conectado debe activarse e inicializarse.
>
> ![Linux][Logo_Linux] Linux
>
> Si los discos están conectados, tiene que iniciar sesión en la máquina virtual e inicializar los discos, como se describe en [este artículo][virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux]
>
>

---
Si el nuevo disco es un disco vacío, también debe dar formato al disco. Para dar formato, especialmente en el caso de los archivos de datos y de registro de DBMS, se aplican las mismas recomendaciones en cuanto a implementaciones sin sistema operativo del DBMS.

Una cuenta de Azure Storage no proporciona recursos infinitos en términos de volumen de E/S, IOPS y volumen de datos. Por lo general, las máquinas virtuales de DBMS son las que más afectadas se ven por esto. Puede que resulte más recomendable utilizar una cuenta de almacenamiento independiente para cada máquina virtual si tiene que implementar algunas VM con un gran volumen de E/S. De este modo, podrá respetar el límite de volumen de la cuenta de Azure Storage. De lo contrario, tendrá que analizar cómo puede asignar estas máquinas virtuales a distintas cuentas de almacenamiento sin superar el límite de cada una de ellas. En la [Guía de implementación de DBMS][dbms-guide] podrá encontrar más información. También debe tener en cuenta estas limitaciones para máquinas virtuales de servidor de aplicaciones de SAP puras u otras máquinas virtuales que podrían terminar por necesitar discos duros virtuales adicionales. Estas restricciones no se aplican si usa un disco de Managed Disks. Si pretende usar Premium Storage, se recomienda utilizar discos de Managed Disks.

Otro tema pertinente para las cuentas de almacenamiento es si los discos duros virtuales de una cuenta de almacenamiento van a tener replicación geográfica. La replicación geográfica se habilita o deshabilita en el nivel de cuenta de almacenamiento, y no en el de máquina virtual. Si la replicación geográfica está habilitada, los VHD incluidos en la cuenta de almacenamiento podrían replicarse en otro centro de datos de Azure de la misma región. Antes tomar una decisión al respecto, debería considerar la siguiente restricción:

La replicación geográfica de Azure funciona de forma local en cada disco duro virtual de una máquina virtual y no replica las E/S por orden cronológico en varios VHD de una VM. Por lo tanto, el disco duro virtual que representa la máquina virtual base, así como cualquier VHD adicional asociado a la VM, se replican sin tener en cuenta los demás. Es decir, los cambios de los distintos VHD no están sincronizados. El hecho de que las E/S se replican independientemente del orden en el que se escriben significa que la replicación geográfica no aporta ninguna ventaja para los servidores de bases de datos que tienen sus bases de datos distribuidas en varios VHD. Además del DBMS, también puede haber otras aplicaciones en las que los procesos escriben o manipulan datos en diferentes VHD y donde es importante mantener el orden de los cambios. Si esto último figura como requisito, no debe habilitarse la replicación geográfica en Azure. En función de si necesita o desea la habilitar la replicación geográfica para un conjunto de máquinas virtuales, pero no para otro conjunto, ya puede categorizar las VM y sus VHD relacionados en distintas cuentas de almacenamiento que tengan la replicación geográfica habilitada o deshabilitada.

#### <a name="setting-automount-for-attached-disks"></a><a name="17e0d543-7e8c-4160-a7da-dd7117a1ad9d"></a>Configuración de montaje automático para discos conectados
---
> ![Windows][Logo_Windows] Windows
>
> En el caso de las máquinas virtuales que se crean a partir de imágenes o discos propios, hay que comprobar y, posiblemente, establecer el parámetro automount. Si se establece este parámetro, la máquina virtual podrá, tras un reinicio o una nueva implementación en Azure, volver a montar automáticamente las unidades asociadas o montadas.
> El parámetro ya está establecido en las imágenes que proporciona Microsoft en Azure Marketplace.
>
> Para configurar el montaje automático, consulte la documentación del ejecutable de línea de comandos diskpart.exe aquí:
>
> * [Opciones de línea de comandos de DiskPart](/previous-versions/windows/it-pro/windows-xp/bb490893(v=technet.10))
> * [Automount (Montaje automático)](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc753703(v=ws.11))
>
> Se debe abrir la ventana de línea de comandos de Windows como administrador.
>
> Si los discos se han adjuntado, tiene que iniciar sesión en la máquina virtual para abrir el administrador de discos de Windows. Si el montaje automático no está habilitado como se recomienda en el capítulo [Configuración de montaje automático para discos conectados][planning-guide-5.5.3], el volumen recién conectado debe activarse e inicializarse.
>
> ![Linux][Logo_Linux] Linux
>
> Debe inicializar un disco vacío recién conectado, como se describe en [este artículo][virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux].
> También debe agregar discos nuevos a /etc/fstab.
>
>

---
### <a name="final-deployment"></a>Implementación final

Para más información acerca de la implementación final y los pasos exactos, especialmente en relación con la implementación de la extensión de Azure para SAP, consulte la [Guía de implementación][deployment-guide].

## <a name="accessing-sap-systems-running-within-azure-vms"></a>Acceso a sistemas SAP que se ejecutan en máquinas virtuales de Azure

Para escenarios donde quiere conectarse con esos sistemas SAP a través de la red pública de Internet mediante la GUI de SAP, deben aplicarse los siguientes procedimientos.

Más adelante en el documento, se verá el otro escenario principal, la conexión a sistemas SAP en implementaciones entre locales que tienen una conexión de sitio a sitio (túnel VPN) o conexión de Azure ExpressRoute entre los sistemas locales y los de Azure.

### <a name="remote-access-to-sap-systems"></a>Acceso remoto a sistemas SAP

Con Azure Resource Manager no hay puntos de conexión predeterminados, como sucedía en el modelo clásico anterior. Todos los puertos de una máquina virtual de Azure Resource Manager están abiertos siempre que se cumplan estos requisitos:

1. No haya ningún grupo de seguridad de red definido para la subred o la interfaz de red. El tráfico de red a las máquinas virtuales de Azure puede protegerse mediante los denominados "grupos de seguridad de red". Para más información, consulte [¿Qué es un grupo de seguridad de red?][virtual-networks-nsg]
2. No se haya definido ningún Azure Load Balancer para la interfaz de red.   

Consulte la diferencia de arquitectura entre el modelo clásico y el de ARM, que se describe en [este artículo][virtual-machines-azure-resource-manager-architecture].

#### <a name="configuration-of-the-sap-system-and-sap-gui-connectivity-over-the-internet"></a>Configuración del sistema SAP y la conectividad de la GUI de SAP a través de Internet

Para obtener información más detallada sobre este tema consulte este artículo: <https://blogs.msdn.com/b/saponsqlserver/archive/2014/06/24/sap-gui-connection-closed-when-connecting-to-sap-system-in-azure.aspx>

#### <a name="changing-firewall-settings-within-vm"></a>Cambio de la configuración del firewall en la máquina virtual

Puede que haga falta configurar el firewall en las máquinas virtuales para permitir el tráfico entrante al sistema SAP.

---
> ![Windows][Logo_Windows] Windows
>
> De forma predeterminada, el Firewall de Windows está activado en una máquina virtual implementada de Azure. Ahora debe permitir la apertura del puerto de SAP; de lo contrario, la GUI de SAP no podrá conectarse.
> Para ello, siga estos pasos:
>
> * Abra Panel de control\Sistema y seguridad\Firewall de Windows y vaya a **Configuración avanzada**.
> * Ahora haga clic con el botón derecho en Reglas de entrada y seleccione **Nueva regla**.
> * En el asistente que aparece a continuación, elija crear una nueva regla de **Puerto**.
> * En el paso siguiente del asistente, deje marcada la opción TCP y escriba el número del puerto que desea abrir. Puesto que el identificador de instancia de SAP es 00, utilizamos 3200. Si la instancia tiene un número de instancia distinto, deberá abrir el puerto que definió anteriormente según el número de instancia.
> * En la siguiente ventana del asistente, tendrá que dejar marcado el elemento **Permitir la conexión**.
> * En el siguiente paso, tendrá que definir si la regla se aplica a redes de tipo Dominio, Privado o Público. Ajuste las opciones necesarias para satisfacer sus necesidades. Sin embargo, si va a conectarse a la GUI de SAP desde el exterior a través de la red pública, tendrá que tener la regla aplicada a dicho tipo de red.
> * En el último paso del asistente, asigne el nombre a la regla y presione **Finalizar** para guardarlo.
>
> La regla se empieza a aplicar de inmediato.
>
> ![Definición de la regla de puertos][planning-guide-figure-1600]
>
> ![Linux][Logo_Linux] Linux
>
> Las imágenes de Linux de Azure Marketplace no habilitan el firewall iptables de forma predeterminada, por lo que la conexión a su sistema SAP debe funcionar. Si habilitó iptables u otro firewall, consulte la documentación de iptables o del firewall utilizado para permitir el tráfico TCP entrante al puerto 32xx (donde xx es el número de su sistema SAP).
>
>

---
#### <a name="security-recommendations"></a>Recomendaciones de seguridad

La GUI de SAP no se conecta inmediatamente a cualquiera de las instancias de SAP (puerto 32xx) que se están ejecutando, sino que primero se conecta mediante el puerto abierto para el proceso de servidor de mensajes de SAP (puerto 36xx). Anteriormente, el servidor de mensajes usaba el mismo puerto para la comunicación interna con las instancias de la aplicación. Para evitar que los servidores de aplicaciones locales se comuniquen por accidente con un servidor de mensajes en Azure, es posible cambiar los puertos de comunicación internos. Se recomienda encarecidamente cambiar la comunicación interna entre el servidor de mensajes de SAP y sus instancias de aplicación a un número de puerto diferente en los sistemas que se hayan clonado a partir de sistemas locales, como un clon de desarrollo para pruebas de proyectos, entre otros casos. Esto puede hacerse con el parámetro de perfil predeterminado:

> rdisp/msserv_internal
>
>

como se documenta en [Security Settings for the SAP Message Server ](https://help.sap.com/saphelp_nwpi71/helpdata/en/47/c56a6938fb2d65e10000000a42189c/content.htm) (Configuración de seguridad del servidor de mensajes de SAP)


### <a name="single-vm-with-sap-netweaver-demotraining-scenario"></a><a name="3e9c3690-da67-421a-bc3f-12c520d99a30"></a>Escenario de demostración/aprendizaje de máquina virtual única con SAP NetWeaver

![Ejecución de sistemas de demostración de SAP de una sola máquina virtual con los mismos nombres de VM, de forma aislada en Azure Cloud Services][planning-guide-figure-1700]

En este caso, se va a implementar un escenario habitual de sistema de demostración o aprendizaje en el que dicho escenario se encuentra por completo en una sola VM. Se supone que la implementación se realiza mediante plantillas de imagen de máquina virtual. También se supone que se deben implementar varias de estas máquinas virtuales de demostración o aprendizaje, y que estas tendrán el mismo nombre. Los sistemas de entrenamiento completo no tienen conectividad a los recursos locales y son lo opuesto a una implementación híbrida.

La suposición es que ha creado una imagen de máquina virtual como se describe en algunas secciones del capítulo [Preparación de máquinas virtuales con SAP para Azure][planning-guide-5.2] de este documento.

La secuencia de eventos para implementar el escenario es similar a esta:

##### <a name="powershell"></a>PowerShell

* Cree un grupo de recursos para cada entorno de aprendizaje o demostración.

```powershell
$rgName = "SAPERPDemo1"
New-AzResourceGroup -Name $rgName -Location "North Europe"
```

* Cree una cuenta de almacenamiento si no desea usar discos de Managed Disks.

```powershell
$suffix = Get-Random -Minimum 100000 -Maximum 999999
$account = New-AzStorageAccount -ResourceGroupName $rgName -Name "saperpdemo$suffix" -SkuName Standard_LRS -Kind "Storage" -Location "North Europe"
```

* Cree una nueva red virtual para cada entorno de aprendizaje o demostración a fin de habilitar el uso del mismo nombre de host y las mismas direcciones IP. La red virtual está protegida por un grupo de seguridad de red que solo permite el tráfico dirigido al puerto 3389 para habilitar el acceso de escritorio remoto y al puerto 22 para SSH.

```powershell
# Create a new Virtual Network
$rdpRule = New-AzNetworkSecurityRuleConfig -Name SAPERPDemoNSGRDP -Protocol * -SourcePortRange * -DestinationPortRange 3389 -Access Allow -Direction Inbound -SourceAddressPrefix * -DestinationAddressPrefix * -Priority 100
$sshRule = New-AzNetworkSecurityRuleConfig -Name SAPERPDemoNSGSSH -Protocol * -SourcePortRange * -DestinationPortRange 22 -Access Allow -Direction Inbound -SourceAddressPrefix * -DestinationAddressPrefix * -Priority 101
$nsg = New-AzNetworkSecurityGroup -Name SAPERPDemoNSG -ResourceGroupName $rgName -Location  "North Europe" -SecurityRules $rdpRule,$sshRule

$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name Subnet1 -AddressPrefix  10.0.1.0/24 -NetworkSecurityGroup $nsg
$vnet = New-AzVirtualNetwork -Name SAPERPDemoVNet -ResourceGroupName $rgName -Location "North Europe"  -AddressPrefix 10.0.1.0/24 -Subnet $subnetConfig
```

* Cree una nueva dirección IP pública que se pueda usar para acceder a la máquina virtual desde Internet.

```powershell
# Create a public IP address with a DNS name
$pip = New-AzPublicIpAddress -Name SAPERPDemoPIP -ResourceGroupName $rgName -Location "North Europe" -DomainNameLabel $rgName.ToLower() -AllocationMethod Dynamic
```

* Cree una nueva interfaz de red para la máquina virtual.

```powershell
# Create a new Network Interface
$nic = New-AzNetworkInterface -Name SAPERPDemoNIC -ResourceGroupName $rgName -Location "North Europe" -Subnet $vnet.Subnets[0] -PublicIpAddress $pip
```

* Cree una máquina virtual. Para este escenario, todas las VM tendrán el mismo nombre. El SID de SAP de las instancias de SAP NetWeaver de esas máquinas virtuales será el mismo. Dentro de un grupo de recursos de Azure, el nombre de la máquina virtual debe ser único, pero puede ejecutar máquinas virtuales con el mismo nombre en diferentes grupos de recursos de Azure. La cuenta "Administrador" predeterminada de Windows o "root" de Linux no son válidas. Por lo tanto, se debe definir un nuevo nombre de usuario administrador, junto con una contraseña. También debe definirse el tamaño de la máquina virtual.

```powershell
#####
# Create a new virtual machine with an official image from the Azure Marketplace
#####
$cred=Get-Credential -Message "Type the name and password of the local administrator account."
$vmconfig = New-AzVMConfig -VMName SAPERPDemo -VMSize Standard_D11

# select image
$vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "MicrosoftWindowsServer" -Offer "WindowsServer" -Skus "2012-R2-Datacenter" -Version "latest"
$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Windows -ComputerName "SAPERPDemo" -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
# $vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "SUSE" -Offer "SLES-SAP" -Skus "12-SP1" -Version "latest"
# $vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "RedHat" -Offer "RHEL" -Skus "7.2" -Version "latest"
# $vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "Oracle" -Offer "Oracle-Linux" -Skus "7.2" -Version "latest"
# $vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Linux -ComputerName "SAPERPDemo" -Credential $cred

$vmconfig = Add-AzVMNetworkInterface -VM $vmconfig -Id $nic.Id

$vmconfig = Set-AzVMBootDiagnostics -Disable -VM $vmconfig
$vm = New-AzVM -ResourceGroupName $rgName -Location "North Europe" -VM $vmconfig
```

```powershell
#####
# Create a new virtual machine with a VHD that contains the private image that you want to use
#####
$cred=Get-Credential -Message "Type the name and password of the local administrator account."
$vmconfig = New-AzVMConfig -VMName SAPERPDemo -VMSize Standard_D11

$vmconfig = Add-AzVMNetworkInterface -VM $vmconfig -Id $nic.Id

$diskName="osfromimage"
$osDiskUri=$account.PrimaryEndpoints.Blob.ToString() + "vhds/" + $diskName  + ".vhd"

$vmconfig = Set-AzVMOSDisk -VM $vmconfig -Name $diskName -VhdUri $osDiskUri -CreateOption fromImage -SourceImageUri <path to VHD that contains the OS image> -Windows
$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Windows -ComputerName "SAPERPDemo" -Credential $cred
#$vmconfig = Set-AzVMOSDisk -VM $vmconfig -Name $diskName -VhdUri $osDiskUri -CreateOption fromImage -SourceImageUri <path to VHD that contains the OS image> -Linux
#$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Linux -ComputerName "SAPERPDemo" -Credential $cred

$vmconfig = Set-AzVMBootDiagnostics -Disable -VM $vmconfig
$vm = New-AzVM -ResourceGroupName $rgName -Location "North Europe" -VM $vmconfig
```

```powershell
#####
# Create a new virtual machine with a Managed Disk Image
#####
$cred=Get-Credential -Message "Type the name and password of the local administrator account."
$vmconfig = New-AzVMConfig -VMName SAPERPDemo -VMSize Standard_D11

$vmconfig = Add-AzVMNetworkInterface -VM $vmconfig -Id $nic.Id

$vmconfig = Set-AzVMSourceImage -VM $vmconfig -Id <Id of Managed Disk Image>
$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Windows -ComputerName "SAPERPDemo" -Credential $cred
#$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Linux -ComputerName "SAPERPDemo" -Credential $cred

$vmconfig = Set-AzVMBootDiagnostics -Disable -VM $vmconfig
$vm = New-AzVM -ResourceGroupName $rgName -Location "North Europe" -VM $vmconfig
```

* Opcionalmente, agregue discos adicionales y restaure el contenido necesario. Todos los nombres de blob (direcciones URL para acceder a los blobs) tienen que ser únicos dentro de Azure.

```powershell
# Optional: Attach additional VHD data disks
$vm = Get-AzVM -ResourceGroupName $rgName -Name SAPERPDemo
$dataDiskUri = $account.PrimaryEndpoints.Blob.ToString() + "vhds/datadisk.vhd"
Add-AzVMDataDisk -VM $vm -Name datadisk -VhdUri $dataDiskUri -DiskSizeInGB 1023 -CreateOption empty | Update-AzVM

# Optional: Attach additional Managed Disks
$vm = Get-AzVM -ResourceGroupName $rgName -Name SAPERPDemo
Add-AzVMDataDisk -VM $vm -Name datadisk -DiskSizeInGB 1023 -CreateOption empty -Lun 0 | Update-AzVM
```

##### <a name="cli"></a>CLI

El siguiente código de ejemplo puede utilizarse en Linux. En el caso de Windows, use PowerShell como se describe arriba o adapte el ejemplo para usar %rgName% en lugar de $rgName y establezca la variable de entorno con el comando de Windows *set*.

* Cree un grupo de recursos para cada entorno de aprendizaje o demostración.

```azurecli
rgName=SAPERPDemo1
rgNameLower=saperpdemo1
az group create --name $rgName --location "North Europe"
```

* Creación de una cuenta de almacenamiento nueva

```azurecli
az storage account create --resource-group $rgName --location "North Europe" --kind Storage --sku Standard_LRS --name $rgNameLower
```

* Cree una nueva red virtual para cada entorno de aprendizaje o demostración a fin de habilitar el uso del mismo nombre de host y las mismas direcciones IP. La red virtual está protegida por un grupo de seguridad de red que solo permite el tráfico dirigido al puerto 3389 para habilitar el acceso de escritorio remoto y al puerto 22 para SSH.

```azurecli
az network nsg create --resource-group $rgName --location "North Europe" --name SAPERPDemoNSG
az network nsg rule create --resource-group $rgName --nsg-name SAPERPDemoNSG --name SAPERPDemoNSGRDP --protocol \* --source-address-prefix \* --source-port-range \* --destination-address-prefix \* --destination-port-range 3389 --access Allow --priority 100 --direction Inbound
az network nsg rule create --resource-group $rgName --nsg-name SAPERPDemoNSG --name SAPERPDemoNSGSSH --protocol \* --source-address-prefix \* --source-port-range \* --destination-address-prefix \* --destination-port-range 22 --access Allow --priority 101 --direction Inbound

az network vnet create --resource-group $rgName --name SAPERPDemoVNet --location "North Europe" --address-prefixes 10.0.1.0/24
az network vnet subnet create --resource-group $rgName --vnet-name SAPERPDemoVNet --name Subnet1 --address-prefix 10.0.1.0/24 --network-security-group SAPERPDemoNSG
```

* Cree una nueva dirección IP pública que se pueda usar para acceder a la máquina virtual desde Internet.

```azurecli
az network public-ip create --resource-group $rgName --name SAPERPDemoPIP --location "North Europe" --dns-name $rgNameLower --allocation-method Dynamic
```

* Cree una nueva interfaz de red para la máquina virtual.

```azurecli
az network nic create --resource-group $rgName --location "North Europe" --name SAPERPDemoNIC --public-ip-address SAPERPDemoPIP --subnet Subnet1 --vnet-name SAPERPDemoVNet
```

* Cree una máquina virtual. Para este escenario, todas las VM tendrán el mismo nombre. El SID de SAP de las instancias de SAP NetWeaver de esas máquinas virtuales será el mismo. Dentro de un grupo de recursos de Azure, el nombre de la máquina virtual debe ser único, pero puede ejecutar máquinas virtuales con el mismo nombre en diferentes grupos de recursos de Azure. La cuenta "Administrador" predeterminada de Windows o "root" de Linux no son válidas. Por lo tanto, se debe definir un nuevo nombre de usuario administrador, junto con una contraseña. También debe definirse el tamaño de la máquina virtual.

```azurecli
#####
# Create virtual machines using storage accounts
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image SUSE:SLES-SAP:12-SP1:latest --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image RedHat:RHEL:7.2:latest --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image "Oracle:Oracle-Linux:7.2:latest" --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --authentication-type password

#####
# Create virtual machines using Managed Disks
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image SUSE:SLES-SAP:12-SP1:latest --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image RedHat:RHEL:7.2:latest --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image "Oracle:Oracle-Linux:7.2:latest" --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --authentication-type password
```

```azurecli
#####
# Create a new virtual machine with a VHD that contains the private image that you want to use
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --os-type Windows --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --image <path to image vhd>
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --os-type Linux --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --image <path to image vhd> --authentication-type password

#####
# Create a new virtual machine with a Managed Disk Image
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --image <managed disk image id>
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --image <managed disk image id> --authentication-type password
```

* Opcionalmente, agregue discos adicionales y restaure el contenido necesario. Todos los nombres de blob (direcciones URL para acceder a los blobs) tienen que ser únicos dentro de Azure.

```azurecli
# Optional: Attach additional VHD data disks
az vm unmanaged-disk attach --resource-group $rgName --vm-name SAPERPDemo --size-gb 1023 --vhd-uri https://$rgNameLower.blob.core.windows.net/vhds/data.vhd  --new

# Optional: Attach additional Managed Disks
az vm disk attach --resource-group $rgName --vm-name SAPERPDemo --size-gb 1023 --disk datadisk --new
```

##### <a name="template"></a>Plantilla

Puede usar las plantillas de ejemplo del repositorio Azure-quickstart-templates en GitHub.

* [Máquina virtual de Linux simple](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-linux)
* [Máquina virtual de Windows simple](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows)
* [Máquina virtual a partir de imagen](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image)

### <a name="implement-a-set-of-vms-that-communicate-within-azure"></a>Implementación de un conjunto de máquinas virtuales que se comunican dentro de Azure

Este caso no híbrido es un escenario habitual para fines de aprendizaje y demostración, en el que el software que representa el escenario de aprendizaje o demostración se divide entre varias VM. Los diferentes componentes instalados en las distintas máquinas virtuales deben comunicarse entre sí. Una vez más, en este escenario no se requiere ninguna comunicación de red local ni ningún escenario entre locales.

Este escenario es una extensión de la instalación descrita en el capítulo [Escenario de demostración/aprendizaje de máquina virtual única con SAP NetWeaver][planning-guide-7.1] de este documento. En este caso, se agregarán más máquinas virtuales a un grupo de recursos existente. En el ejemplo siguiente el entorno de aprendizaje consta de una máquina virtual de SAP ASCS/SCS, una máquina virtual que ejecuta un DBMS y otra de instancia del servidor de aplicaciones de SAP.

Antes de poner en práctica este escenario, tendrá que considerar la configuración básica, como ya se hizo en el escenario anterior.

#### <a name="resource-group-and-virtual-machine-naming"></a>Nomenclatura de los grupos de recursos y las máquinas virtuales

Todos los nombres de los grupos de recursos deben ser únicos. Desarrolle su propio esquema de nomenclatura de los recursos como, por ejemplo, `<rg-name`>-sufijo.

El nombre de la máquina virtual tiene que ser único dentro del grupo de recursos.

#### <a name="set-up-network-for-communication-between-the-different-vms"></a>Configuración de la red para la comunicación entre las distintas máquinas virtuales

![Conjunto de máquinas virtuales dentro de una instancia de Azure Virtual Network][planning-guide-figure-1900]

Para evitar colisiones de nomenclatura con clones de los mismos entornos de aprendizaje o demostración, deberá crear una instancia de Azure Virtual Network para cada entorno. La resolución de nombres DNS correrá a cargo de Azure o podrá configurar su propio servidor DNS fuera de Azure (sobre lo cual no se proporcionarán más detalles en el presente artículo). En este escenario, no se debe configurar un DNS propio. La comunicación a través de los nombres de host se habilitará para todas las máquinas virtuales dentro de Azure Virtual Network.

Los motivos para separar los entornos de aprendizaje o demostración por redes virtuales en lugar de por grupos de recursos pueden ser los siguientes:

* Tal y como está configurado, el entorno de SAP necesita su propio AD/OpenLDAP y un servidor de dominio debe formar parte de cada uno de los entornos.  
* Tal y como está configurado, el entorno de SAP tiene componentes que deben funcionar con direcciones IP fijas.

Se puede encontrar información más detallada sobre Azure Virtual Network y cómo definirlas en [este artículo][virtual-networks-create-vnet-arm-pportal].

## <a name="deploying-sap-vms-with-corporate-network-connectivity-cross-premises"></a>Implementación de VM de SAP con conectividad de red corporativa (entre entornos locales)

Está ejecutando un entorno de SAP y desea dividir la implementación entre equipos sin sistema operativo para servidores de DBMS de gama alta, entornos locales virtualizados para capas de aplicaciones, y sistemas SAP e IaaS de Azure de menor tamaño configurados en dos niveles. El supuesto básico es que los sistemas SAP dentro de un entorno de SAP deben comunicarse entre sí y con muchos otros componentes de software implementados en la empresa, independientemente del modo de implementación. El modo de implementación tampoco debería introducir ninguna diferencia para el usuario final que se conecta con la GUI de SAP u otras interfaces. Estas condiciones solo pueden cumplirse cuando se dispone de servicios de Active Directory/OpenLDAP locales y servicios DNS extendidos a los sistemas de Azure por medio de una conectividad de sitio a sitio o de varios sitios, o mediante conexiones privadas como Azure ExpressRoute.



### <a name="scenario-of-an-sap-landscape"></a>Escenario de un entorno de SAP

El escenario entre locales o híbrido se puede describir aproximadamente como se indica en los gráficos siguientes:

![Conectividad de sitio a sitio entre activos locales y de Azure][planning-guide-figure-2100]

El requisito mínimo es el uso de protocolos de comunicaciones seguras como SSL/TLS para el acceso del navegador o conexiones basadas en VPN para el acceso del sistema a los servicios de Azure. Se presupone que las empresas controlan la conexión VPN entre su red corporativa y Azure de forma distinta. Puede que algunas empresas se limiten a abrir todos los puertos. Es posible que otras deseen ser precisas, por ejemplo, en cuanto a los puertos que hay que abrir.

En la tabla siguiente, se enumeran los puertos de comunicación de SAP típicos. Básicamente, basta con abrir el puerto de la puerta de enlace de SAP.

<!-- sapms is prefix of a SAP service name and not a spelling error -->

| Servicio | Nombre del puerto | Ejemplo `<nn`> = 01 | Intervalo predeterminado (mín.-máx.) | Comentario |
| --- | --- | --- | --- | --- |
| Distribuidor |sapdp`<nn>` consulte * |3201 |3200 - 3299 |Distribuidor de SAP, utilizado por la GUI de SAP para Windows y Java |
| Servidor de mensajes |sapms`<sid`> consulte ** |3600 |sapms libre`<anySID`> |sid = identificador del sistema SAP |
| Puerta de enlace |sapgw`<nn`> consulte * |3301 |libre |Puerta de enlace de SAP, utilizada para la comunicación de CPIC y RFC |
| Enrutador de SAP |sapdp99 |3299 |libre |Solo los nombres de servicio de CI (instancia central) se pueden reasignar a /etc/services con un valor arbitrario después de la instalación. |

*) nn = número de instancia de SAP

\*\*) sid = identificador del sistema SAP

Podrá encontrar información más detallada sobre los puertos necesarios para diferentes productos o servicios de productos de SAP aquí <https://scn.sap.com/docs/DOC-17124>.
Con este documento debería poder abrir los puertos dedicados del dispositivo VPN necesarios para escenarios y productos de SAP concretos.

Otra posible medida de seguridad para la implementación de máquinas virtuales en este escenario es la creación de un [grupo de seguridad de red][virtual-networks-nsg] para definir las reglas de acceso.



#### <a name="printing-on-a-local-network-printer-from-sap-instance-in-azure"></a>Impresión en una impresora de red local desde la instancia de SAP en Azure

##### <a name="printing-over-tcpip-in-cross-premises-scenario"></a>Impresión por TCP/IP en el escenario entre locales

La configuración de las impresoras de red basadas en TCP/IP locales en una VM de Azure es generalmente la misma que la de la red corporativa, partiendo de la base de que tenga establecidos un túnel VPN de sitio a sitio o una conexión ExpressRoute.

---
> ![Windows][Logo_Windows] Windows
>
> Para ello, siga estos pasos:
>
> * Algunas impresoras de red incluyen un Asistente para la configuración, lo que permite configurarlas más fácilmente en una VM de Azure. Si no se distribuyó ningún software de asistente con la impresora, la forma manual de configurar la impresora consiste en crear un nuevo puerto de impresora TCP/IP.
> * Abra Panel de control -> Dispositivos e impresoras -> Agregar una impresora.
> * Elija Agregar una impresora por medio de una dirección TCP/IP o un nombre de host.
> * Escriba la dirección IP de la impresora.
> * Puerto de impresora estándar 9100.
> * Si es necesario, instale manualmente el controlador de impresora adecuado.
>
> ![Linux][Logo_Linux] Linux
>
> * Como en Windows, solo tiene que seguir el procedimiento estándar para la instalación de una impresora de red.
> * Solo siga las guías de Linux públicas de [SUSE](https://www.suse.com/documentation/sles-12/book_sle_deployment/data/sec_y2_hw_print.html) o [Red Hat y Oracle Linux](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/sec-Printer_Configuration.html) sobre cómo agregar una impresora.
>
>

---
![Impresión en red][planning-guide-figure-2200]

##### <a name="host-based-printer-over-smb-shared-printer-in-cross-premises-scenario"></a>Impresora basada en host por SMB (impresora compartida) en el escenario entre locales

Las impresoras basadas en host están diseñadas para no ser compatibles con la red. No obstante, una impresora basada en host se puede compartir entre los equipos de una red si la impresora está conectada a un equipo encendido. Conecte la red corporativa sitio a sitio o por ExpressRoute y comparta la impresora local. El protocolo SMB usa NetBIOS en lugar de DNS como servicio de nombres. El nombre de host de NetBIOS puede ser diferente del nombre de host de DNS. Lo normal es que el nombre de host de NetBIOS y el nombre de host de DNS sean idénticos. El dominio de DNS no tiene sentido en el espacio de nombres de NetBIOS. En consecuencia, el nombre de host de DNS completo que consta del nombre de host de DNS y el dominio de DNS no debe utilizarse en el espacio de nombres de NetBIOS.

El recurso compartido de impresora se identifica mediante un nombre único en la red:

* Nombre de host del host de SMB (se requiere siempre).
* Nombre del recurso compartido (se requiere siempre).
* Nombre del dominio si el recurso compartido de impresora no está en el mismo dominio que el sistema SAP.
* Además, puede que se requieran un nombre de usuario y una contraseña para acceder al recurso compartido de impresora.

Procedimientos:

---
> ![Windows][Logo_Windows] Windows
>
> Comparta la impresora local.
> En la máquina virtual de Azure, abra el Explorador de Windows y escriba el nombre del recurso compartido de impresora.
> El Asistente para la instalación de la impresora le guiará a través del proceso de instalación.
>
> ![Linux][Logo_Linux] Linux
>
> A continuación, se ofrecen algunos ejemplos de documentación sobre la configuración de impresoras de red en Linux e incluso un capítulo sobre la impresión en Linux. Funcionará del mismo modo que en una máquina virtual de Linux en Azure si la máquina virtual forma parte de una VPN:
>
> * SLES <https://en.opensuse.org/SDB:Printing_via_SMB_(Samba)_Share_or_Windows_Share>
> * RHEL u Oracle Linux <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sec-Printer_Configuration.html#s1-printing-smb-printer>
>
>

---
##### <a name="usb-printer-printer-forwarding"></a>Impresora USB (reenvío de impresora)

En Azure, la capacidad de los Servicios de Escritorio remoto para permitir a los usuarios que accedan a sus dispositivos de impresora local en una sesión remota no está disponible.

---
> ![Windows][Logo_Windows] Windows
>
> Se puede encontrar más información sobre la impresión con Windows aquí: <https://technet.microsoft.com/library/jj590748.aspx>.
>
>

---
#### <a name="integration-of-sap-azure-systems-into-correction-and-transport-system-tms-in-cross-premises"></a>Integración de sistemas SAP de Azure en el sistema de corrección y transporte (TMS) en el escenario entre locales

Se debe configurar Change and Transport System (TMS) de SAP para exportar las solicitudes de transporte en todos los sistemas del entorno e importarlas de ellos. Se presupone que las instancias de desarrollo de un sistema SAP (de desarrollo) se encuentran en Azure, mientras que el control de calidad (QA) y los sistemas productivos (PRD) son locales. Además, se presupone que hay un directorio central de transporte.

##### <a name="configuring-the-transport-domain"></a>Configuración del dominio de transporte

Configure el dominio de transporte en el sistema que designó como controlador de dominio de transporte según lo descrito en [Configuring the Transport Domain Controller](https://help.sap.com/erp2005_ehp_04/helpdata/en/44/b4a0b47acc11d1899e0000e829fbbd/content.htm)(Configuración del controlador del dominio de transporte). Se creará un TMSADM de usuarios del sistema y se generará el destino requerido de la RFC. Puede comprobar estas conexiones RFC mediante el SM59 de transacciones. La resolución de nombre de host debe estar habilitada en todo el dominio de transporte.

Procedimientos:

* En nuestro escenario, hemos decidido que el sistema de control de calidad local será el controlador de dominio de CTS. Llame al STMS de transacciones. Aparecerá el cuadro de diálogo de TMS. Se mostrará un cuadro de diálogo Configure Transport Domain (Configurar dominio de transporte). (Este cuadro de diálogo aparecerá solo si todavía no ha configurado un dominio de transporte).
* Asegúrese de que el TMSADM de usuario creado automáticamente está autorizado (SM59 -> ABAP Connection [Conexión ABAP] -> TMSADM@E61.DOMAIN_E61 -> Details [Detalles] -> Utilities(M) [Utilidades (M)] -> Authorization Test [Prueba de autorización]). La pantalla inicial del STMS de transacciones debería mostrar que este sistema SAP ahora funciona como el controlador del dominio de transporte, tal y como se muestra a continuación:

![Pantalla inicial del STMS de transacciones en el controlador de dominio][planning-guide-figure-2300]

#### <a name="including-sap-systems-in-the-transport-domain"></a>Inclusión de los sistemas SAP en el dominio de transporte

La secuencia de inclusión de un sistema SAP en un dominio de transporte tiene el aspecto siguiente:

* En el sistema de desarrollo de Azure, vaya al sistema de transporte (cliente 000) y llame al STMS de transacciones. Elija Other Configuration (Otra configuración) en el cuadro de diálogo y continúe con Include System in Domain (Incluir sistema en el dominio). Especifique el controlador de dominio como host de destino [[Including SAP Systems in the Transport Domain](https://help.sap.com/erp2005_ehp_04/helpdata/en/44/b4a0c17acc11d1899e0000e829fbbd/content.htm?frameset=/en/44/b4a0b47acc11d1899e0000e829fbbd/frameset.htm)(Inclusión de los sistemas SAP en el dominio de transporte)]. El sistema se pondrá en espera de que se le incluya en el dominio de transporte.
* Por motivos de seguridad, deberá volver al controlador de dominio para confirmar la solicitud. Elija System Overview (Información general del sistema) y Approve of the waiting system (Aprobar el sistema en espera). A continuación, confirme el aviso y la configuración se distribuirá.

Este sistema SAP ahora contiene la información necesaria sobre todos los demás sistemas SAP en el dominio de transporte. Al mismo tiempo, los datos de la dirección del nuevo sistema SAP se envían a todos los demás sistemas SAP y el sistema SAP se introduce en el perfil de transporte del programa de control de transporte. Compruebe si las RFC y el acceso al directorio de transporte del dominio funcionan.

Continúe con la configuración del sistema de transporte como de costumbre según lo descrito en la documentación [Change and Transport System](https://help.sap.com/saphelp_nw70ehp3/helpdata/en/48/c4300fca5d581ce10000000a42189c/content.htm?frameset=/en/44/b4a0b47acc11d1899e0000e829fbbd/frameset.htm)(Sistema de cambios y transportes).

Procedimientos:

* Asegúrese de que su STMS local está configurado correctamente.
* Asegúrese de que la máquina virtual en Azure puede resolver el nombre de host del controlador de dominio de transporte y viceversa.
* Call transaction STMS (Llamar a STMS de transacciones) -> Other Configuration (Otra configuración) -> Include System in Domain (Incluir sistema en el dominio).
* Confirme la conexión en el sistema de TMS local.
* Configure las rutas, grupos y capas de transporte como de costumbre.

En escenarios conectados entre locales conectados de sitio a sitio, la latencia entre el entorno local y el de Azure todavía puede ser considerable. Si sigue la secuencia del transporte de objetos a través de los sistemas de desarrollo y pruebas hasta la producción o se plantea la aplicación de paquetes de transporte o de soporte a los distintos sistemas, se dará cuenta de que, en función de la ubicación del directorio central de transporte, algunos de los sistemas experimentarán una alta latencia al leer o escribir datos en dicho directorio. La situación es similar a las configuraciones en el entorno de SAP, donde los distintos sistemas se propagan a través de diversos centros de datos con una distancia considerable entre los centros de datos.

Para solucionar tal latencia y que los sistemas funcionen con rapidez para las tareas de lectura o escritura o desde el directorio de transporte, puede configurar dos dominios de transporte STMS (uno para los sistemas locales y otro con los sistemas en Azure) y vincular los dominios de transporte. Revise esta documentación que explica los principios que se esconden tras este concepto en el TMS de SAP: <https://help.sap.com/saphelp_me60/helpdata/en/c4/6045377b52253de10000009b38f889/content.htm?frameset=/en/57/38dd924eb711d182bf0000e829fbfe/frameset.htm>.

Procedimientos:

* Configurar un dominio de transporte en cada ubicación (local y Azure) mediante STMS de transacción <https://help.sap.com/saphelp_nw70ehp3/helpdata/en/44/b4a0b47acc11d1899e0000e829fbbd/content.htm>
* Vincular los dominios con un vínculo de dominio y confirmar el vínculo entre ambos.
  <https://help.sap.com/saphelp_nw73ehp1/helpdata/en/a3/139838280c4f18e10000009b38f8cf/content.htm>
* Distribución de la configuración al sistema vinculado.

#### <a name="rfc-traffic-between-sap-instances-located-in-azure-and-on-premises-cross-premises"></a>Tráfico de RFC entre las instancias de SAP ubicadas en Azure y las locales (entre locales)

El tráfico de RFC entre los sistemas en Azure y los locales tiene que funcionar. Para configurar un SM59 de transacciones de llamadas de conexión en un sistema de origen donde se debe definir una conexión RFC hacia el sistema de destino. La configuración es similar a la configuración estándar de una conexión RFC estándar.

Se presupone que en el escenario entre locales, las máquinas virtuales que ejecutan sistemas SAP que tienen que comunicarse entre sí se encuentran en el mismo dominio. Por lo tanto, la configuración de una conexión RFC entre sistemas SAP no difiere de los pasos y las entradas de configuración de los escenarios locales.

#### <a name="accessing-local-fileshares-from-sap-instances-located-in-azure-or-vice-versa"></a>Acceso a recursos compartidos de archivos locales desde instancias de SAP ubicadas en Azure o viceversa

Las instancias de SAP ubicadas en Azure tienen que acceder a los recursos compartidos de archivos que están dentro de los locales corporativos. Además, las instancias de SAP locales tienen que acceder a los recursos compartidos de archivos que se encuentran en Azure. Para habilitar los recursos compartidos de archivos, se tienen que configurar los permisos y las opciones de uso compartido en el sistema local. Asegúrese de abrir los puertos en la conexión VPN o ExpressRoute entre Azure y su centro de datos.

## <a name="supportability"></a>Compatibilidad

### <a name="azure-extension-for-sap"></a><a name="6f0a47f3-a289-4090-a053-2521618a28c3"></a>Extensión de Azure para SAP

A fin de proporcionar parte de la información de infraestructura de Azure de los sistemas SAP críticos a las instancias de agente del host de SAP instaladas en las máquinas virtuales, debe instalarse una extensión de Azure (VM) para SAP en las máquinas virtuales implementadas. Puesto que las demandas de SAP eran muy específicas para las aplicaciones de SAP, Microsoft decidió no implementar de forma genérica la funcionalidad requerida en Azure, y dejar al criterio de los clientes la implementación de las configuraciones y las extensiones de máquina virtual necesarias para sus máquinas virtuales que se ejecutan en Azure. Sin embargo, Azure automatizará la mayor parte de la implementación y la administración del ciclo de vida de la extensión de Azure VM para SAP.

#### <a name="solution-design"></a>Diseño de la solución

La solución desarrollada para permitir al agente del host de SAP obtener la información necesaria se basa en la arquitectura del marco de agente y extensión de Azure VM. La idea del marco de agente y extensión de VM Azure es permitir la instalación de las aplicaciones de software disponibles en la galería de extensiones de VM de Azure dentro de una máquina virtual. La idea principal que subyace tras este concepto es permitir (en casos como el de la extensión de Azure para SAP) la implementación de una funcionalidad especial en una máquina virtual y la configuración de dicho software al realizar la implementación.

El "agente de VM de Azure" que permite controlar extensiones específicas de VM de Azure dentro de la máquina virtual se inyecta en las máquinas virtuales de Windows de forma predeterminada al crear VM en Azure Portal. En el caso de SUSE, Red Hat u Oracle Linux, el agente de VM ya forma parte de la imagen de Azure Marketplace. En caso de que se desee cargar una VM de Linux desde un sistema local a Azure, el agente de VM se debe instalar manualmente.

Los bloques de creación básicos de la solución que proporciona información de la infraestructura de Azure al agente del host de SAP en Azure tienen el siguiente aspecto:

![Componentes de la extensión de Microsoft Azure][planning-guide-figure-2400]

Como se muestra en el diagrama de bloques anterior, una parte de la solución se hospeda en la galería de imágenes de máquina virtual de Azure y de extensiones de Azure (un repositorio replicado globalmente que está administrado por Azure Operations). El equipo conjunto de SAP/Microsoft que trabaja en la implementación de SAP en Azure debe trabajar con las operaciones de Azure para publicar nuevas versiones de la extensión de Azure para SAP.

Cuando se implementa una máquina virtual de Windows nueva, el agente de VM de Azure se agrega automáticamente a la máquina virtual. La función de este agente es coordinar la carga y la configuración de las extensiones de Azure de las máquinas virtuales. En las máquinas virtuales Linux, el agente de máquina virtual de Azure ya forma parte de la imagen de sistema operativo de Azure Marketplace.

Sin embargo, el cliente aún deberá realizar otro paso, que consiste en habilitar y configurar la recopilación de rendimiento. El proceso relacionado con la configuración se automatiza mediante un script de PowerShell o un comando de la CLI. Se puede descargar el script de PowerShell en el Centro de scripts de Microsoft Azure como se describe en la [Guía de implementación][deployment-guide].

La arquitectura general de la extensión de Azure para SAP tiene el siguiente aspecto:

![Extensión de Azure para SAP ][planning-guide-figure-2500]

**Para las instrucciones exactas y los pasos detallados para utilizar estos cmdlets de PowerShell o el comando de la CLI durante las implementaciones, siga las instrucciones de la [Guía de implementación][deployment-guide].**

### <a name="integration-of-azure-located-sap-instance-into-saprouter"></a>Integración de una instancia de SAP ubicada en Azure en SAProuter

Las instancias de SAP que se ejecutan en Azure también deben ser accesibles desde SAProuter.

![Conexión de red SAP-enrutador][planning-guide-figure-2600]

SAProuter permite la comunicación de TCP/IP entre los sistemas participantes si no hay ninguna conexión IP directa. Esto aporta la ventaja de que no se requiere ninguna conexión de un extremo a otro entre los asociados de comunicación en el ámbito de la red. De forma predeterminada, SAProuter escucha en el puerto 3299.
Para conectarse a instancias de SAP por medio de SAProuter, se debe proporcionar la cadena y el nombre de host de SAProuter cada vez que trate de conectarse.

## <a name="sap-netweaver-as-java"></a>AS de SAP NetWeaver en Java

Hasta ahora, este documento se ha centrado en SAP NetWeaver en general o en la pila de ABAP de SAP NetWeaver. En esta pequeña sección, se ofrecen las consideraciones específicas para la pila de Java de SAP. Una de las aplicaciones de SAP NetWeaver basadas exclusivamente en Java más importantes es SAP Enterprise Portal. Otras aplicaciones basadas en SAP NetWeaver como SAP PI y SAP Solution Manager usan las pilas de ABAP y de Java de SAP NetWeaver. Por lo tanto, indudablemente también se deben tener en consideración los aspectos específicos relacionados con la pila de Java de SAP NetWeaver.

### <a name="sap-enterprise-portal"></a>SAP Enterprise Portal

La configuración de un portal de SAP en una máquina virtual de Azure no difiere de la instalación local si se desea implementar escenarios entre locales. Dado que el DNS se ejecuta en el entorno local, la configuración local se puede extrapolar a la de los puertos de las instancias individuales. Las recomendaciones y restricciones descritas hasta ahora en este documento se aplican en una aplicación como SAP Enterprise Portal o la pila de Java de SAP NetWeaver en general.

![Exposición del portal de SAP][planning-guide-figure-2700]

Un escenario de implementación especial de algunos clientes es la exposición directa de SAP Enterprise Portal a Internet mientras el host de máquina virtual está conectado a la red corporativa por medio del túnel VPN de sitio a sitio o ExpressRoute. En semejante escenario, hay que asegurarse de que determinados puertos estén abiertos y no estén bloqueados por el firewall ni por el grupo de seguridad de red. 

El URI inicial del portal es http(s):`<Portalserver`>:5XX00/irj, donde está formado el puerto, tal y como se explica en detalle por SAP en <https://help.sap.com/saphelp_nw70ehp1/helpdata/de/a2/f9d7fed2adc340ab462ae159d19509/frameset.htm>.

![Configuración del punto de conexión][planning-guide-figure-2800]

Si desea personalizar la URL o los puertos de SAP Enterprise Portal, consulte esta documentación:

* [Change Portal URL (Cambio de la dirección URL del portal)](https://wiki.scn.sap.com/wiki/display/EP/Change+Portal+URL)
* [Change Default port numbers, Portal port numbers (Cambio de los números de puerto predeterminados y los números de puerto del portal)](https://wiki.scn.sap.com/wiki/display/NWTech/Change+Default++port+numbers%2C+Portal+port+numbers)

## <a name="high-availability-ha-and-disaster-recovery-dr-for-sap-netweaver-running-on-azure-virtual-machines"></a>Alta disponibilidad (HA) y recuperación ante desastres (DR) para la ejecución de SAP NetWeaver en Azure Virtual Machines

### <a name="definition-of-terminologies"></a>Definición de terminología

El término **alta disponibilidad (HA)** se asocia generalmente a un conjunto de tecnologías que minimiza las interrupciones de TI al proporcionar una continuidad empresarial de los servicios de TI mediante componentes redundantes, tolerantes a errores o protegidos mediante conmutación por error dentro del **mismo** centro de datos. En nuestro caso, dentro de una sola región de Azure.

La **recuperación ante desastres (DR)** también tiene el objetivo de minimizar las interrupciones en los servicios de TI y recuperarlos pero en **distintos** centros de datos que suelen encontrarse a cientos de kilómetros de distancia. En nuestro caso, suelen estar repartidos entre distintas regiones de Azure dentro de la misma región geopolítica o según lo haya establecido usted como cliente.

### <a name="overview-of-high-availability"></a>Información general sobre la alta disponibilidad

Podemos descomponer la explicación sobre la alta disponibilidad de SAP en Azure en dos partes:

* **Alta disponibilidad de la infraestructura de Azure**, por ejemplo, la alta disponibilidad del proceso (VM), la red, el almacenamiento, etc., y sus beneficios para aumentar la disponibilidad de las aplicaciones de SAP.
* **Alta disponibilidad de las aplicaciones de SAP**, por ejemplo, la alta disponibilidad de los componentes de software de SAP:
  * servidores de aplicaciones de SAP,
  * instancia de SAP ASCS/SCS,
  * servidor de base de datos,

y cómo se puede combinar con la alta disponibilidad de la infraestructura de Azure.

La alta disponibilidad de SAP en Azure presenta algunas diferencias con respecto a la alta disponibilidad de SAP en un entorno físico o virtual local. Las siguientes notas del producto de SAP describen las configuraciones estándar de la alta disponibilidad de SAP en entornos virtualizados de Windows: <https://scn.sap.com/docs/DOC-44415>. No hay ninguna configuración de alta disponibilidad de SAP integrada en SAPInst para Linux como la que existe para Windows. Con respecto a la alta disponibilidad de SAP local para Linux, se puede encontrar más información aquí: <https://scn.sap.com/docs/DOC-8541>.

### <a name="azure-infrastructure-high-availability"></a>Alta disponibilidad de la infraestructura de Azure

Actualmente hay un contrato de nivel de servicio del 99,9 % para una única máquina virtual. Para tener una idea de cómo sería la disponibilidad de una sola máquina virtual, puede compilar el producto de los distintos Acuerdos de Nivel de Servicio de Azure disponibles: <https://azure.microsoft.com/support/legal/sla/>.

La base para el cálculo es de30 días por mes o 43 200 minutos. Por lo tanto, un tiempo de inactividad del 0,05 % corresponde a 21,6 minutos. Como de costumbre, la disponibilidad de los distintos servicios se multiplicará de la siguiente manera:

(Servicio de disponibilidad n.º 1/100) * (servicio de disponibilidad n.º 2/100) * (servicio de disponibilidad n.º 3/100) 

Como, por ejemplo:

(99,95/100) * (99,9/100) * (99,9/100) = 0,9975 o una disponibilidad general del 99,75 %.

#### <a name="virtual-machine-vm-high-availability"></a>Alta disponibilidad de la máquina virtual (VM)

Existen dos tipos de eventos de plataforma Azure que pueden afectar a la disponibilidad de sus máquinas virtuales: mantenimiento planeado y mantenimiento no planeado.

* Eventos de mantenimiento planeado son actualizaciones periódicas realizadas por Microsoft en la plataforma Azure subyacente para mejorar en general la fiabilidad, el rendimiento y la seguridad de la infraestructura de la plataforma sobre las que se funcionan sus máquinas virtuales.
* Eventos de mantenimiento no planeado se producen cuando el hardware o la infraestructura física subyacente a su máquina virtual presentan algún tipo de error. Entre estos podemos encontrar errores de la red local, errores de los discos locales u otras errores a nivel de bastidor. Cuando se encuentra un error de este tipo, la plataforma Azure migrará automáticamente la máquina virtual del servidor físico en mal estado que hospeda la máquina virtual a un servidor físico con un estado correcto. Estos eventos no son habituales, pero pueden hacer que su máquina virtual se reinicie.

Se puede encontrar más información en esta documentación: <https://azure.microsoft.com/documentation/articles/virtual-machines-manage-availability>

#### <a name="azure-storage-redundancy"></a>Redundancia de Azure Storage

Los datos de la cuenta de Microsoft Azure Storage se replican siempre para garantizar su durabilidad y alta disponibilidad, en cumplimiento con el contrato de nivel de servicio de Azure Storage, incluso en caso de errores transitorios del hardware.

Dado que Azure Storage mantiene de forma predeterminada tres imágenes de los datos, no se precisan RAID5 o RAID1 en varios discos de Azure.

Se puede encontrar más información en este artículo: <https://azure.microsoft.com/documentation/articles/storage-redundancy/>

#### <a name="utilizing-azure-infrastructure-vm-restart-to-achieve-higher-availability-of-sap-applications"></a>Utilización del reinicio de VM de la infraestructura de Azure para lograr una mayor disponibilidad de las aplicaciones de SAP

Si decide no utilizar funcionalidades como los clústeres de conmutación por error de Windows Server (WSFC) o Pacemaker en Linux (compatible actualmente solo para SLES 12 y versiones posteriores), el reinicio de VM de Azure se utiliza para proteger un sistema SAP contra el tiempo de inactividad, ya esté planeado o no, de la infraestructura de servidores físicos de Azure y la plataforma Azure global subyacente.

> [!NOTE]
> Es importante mencionar que el reinicio de VM de Azure protege principalmente las máquinas virtuales, pero NO las aplicaciones. El reinicio de VM no ofrece una alta disponibilidad en las aplicaciones de SAP, pero sí un cierto grado de disponibilidad de la infraestructura y, por tanto, ofrece indirectamente una mayor disponibilidad de los sistemas SAP. Tampoco hay ningún acuerdo de nivel de servicio para el tiempo que se tarda en reiniciar una máquina virtual después de una interrupción del host planeada o no planeada. Por lo tanto, este método de "alta disponibilidad" no es adecuado para los componentes esenciales de un sistema SAP como (A)SCS o DBMS.
>
>

El almacenamiento es otro elemento infraestructural importante para la alta disponibilidad. Por ejemplo, el Acuerdo de Nivel de Servicio de Azure Storage presenta una disponibilidad del 99,9 %. Si se implementan todas las máquinas virtuales con sus discos en una sola cuenta de Azure Storage, la eventual indisponibilidad de Azure Storage ocasionará que no estén disponibles ninguna de las máquinas virtuales ubicadas en dicha cuenta de almacenamiento ni tampoco los componentes de SAP que se ejecutan dentro de esas máquinas virtuales.  

En lugar de colocar todas las máquinas virtuales en una sola cuenta de Azure Storage, también puede usar cuentas de almacenamiento dedicadas para cada máquina virtual y, de esta manera, aumentar la disponibilidad general de las aplicaciones de SAP al usar varias cuentas de Azure Storage independientes.

Los discos de Azure Managed Disks se colocan automáticamente en el dominio de error de la máquina virtual a la que están conectados. Si coloca dos máquinas virtuales en un conjunto de disponibilidad y usa discos de Managed Disks, la plataforma también se encargará de distribuir estos discos a los diferentes dominios de error. Si pretende usar Premium Storage, se recomienda encarecidamente utilizar también discos de Managed Disks.

La arquitectura de ejemplo de un sistema SAP NetWeaver que utilice la alta disponibilidad de la infraestructura de Azure y las cuentas de almacenamiento podría ser como esta:

![Utilización de la alta disponibilidad de la infraestructura de Azure para lograr una mayor disponibilidad de las aplicaciones de SAP][planning-guide-figure-2900]

La arquitectura de ejemplo de un sistema SAP NetWeaver que utilice la alta disponibilidad de la infraestructura de Azure y los discos de Managed Disks podría ser como esta:

![Utilización de la alta disponibilidad de la infraestructura de Azure para lograr una mayor disponibilidad de las aplicaciones de SAP][planning-guide-figure-2901]

Hasta la fecha, en los componentes esenciales de SAP se ha conseguido lo siguiente:

* Alta disponibilidad de los servidores de aplicaciones (AS) de SAP

  Las instancias de los servidores de aplicaciones de SAP son componentes redundantes. Cada instancia del servidor de aplicaciones de SAP se implementa en su propia máquina virtual, que se ejecuta en otro dominio de error y de actualización de Azure (consulte los capítulos [Dominios de error][planning-guide-3.2.1] y [Dominios de actualización][planning-guide-3.2.2]). Esto se garantiza mediante conjuntos de disponibilidad de Azure (consulte el capítulo [Conjuntos de disponibilidad de Azure][planning-guide-3.2.3]). La eventual indisponibilidad planeada o no de un dominio de error o de actualización de Azure hará que un número limitado de máquinas virtuales y sus respectivas instancias de SAP no estén disponibles.

  Cada instancia del servidor de aplicaciones de SAP se coloca en su propia cuenta de almacenamiento de Azure, por lo que la eventual indisponibilidad de una cuenta de Azure Storage hará que solo deje de estar disponible una máquina virtual y su respectiva instancia de SAP. Sin embargo, tenga en cuenta que las suscripciones de Azure admiten un número de cuentas de Azure Storage limitado. Para garantizar el inicio automático de la instancia de (A)SCS después del reinicio de la máquina virtual, asegúrese de establecer el parámetro Autostart en el perfil de inicio de la instancia de (A)SCS como se indica en el capítulo [Uso de Autostart en las instancias de SAP][planning-guide-11.5].
  Lea también el capítulo [Alta disponibilidad en los servidores de aplicaciones de SAP][planning-guide-11.4.1] para más información.

  Incluso si usa discos de Managed Disks, estos también se almacenan en una cuenta de Azure Storage y pueden no estar disponibles en caso de que se produzca alguna interrupción del almacenamiento.

* *Mayor* disponibilidad de la instancia de SAP (A)SCS

  Aquí se utiliza el reinicio de VM de Azure para proteger la máquina virtual con la instancia de SAP (A)SCS instalada. En el caso de un tiempo de inactividad planeado o no de los servidores de Azure, las máquinas virtuales se reiniciarán en otro servidor disponible. Como se mencionó anteriormente, el reinicio de VM de Azure protege principalmente las VM, pero NO las aplicaciones (en este caso, la instancia de [A]SCS). Mediante el reinicio de máquina virtual, se alcanza indirectamente una mayor disponibilidad de la instancia de SAP (A)SCS. Para garantizar el inicio automático de la instancia de (A)SCS después del reinicio de la máquina virtual, asegúrese de establecer el parámetro Autostart en el perfil de inicio de la instancia de (A)SCS como se indica en el capítulo [Uso de Autostart en las instancias de SAP][planning-guide-11.5]. Esto significa que la instancia de (A)SCS como único punto de error (SPOF) que se ejecuta en una sola máquina virtual será el factor determinante para la disponibilidad de todo el entorno de SAP.

* *Mayor* disponibilidad del servidor de DBMS

  Aquí, de forma similar al caso de uso de la instancia de SAP (A)SCS, se utiliza el reinicio de VM de Azure para proteger la máquina virtual con el software de DBMS instalado, y se alcanza una mayor disponibilidad del software de DBMS al reiniciar la máquina virtual.
  El DBMS que se ejecute en una sola máquina virtual también es un SPOF, por lo que es el factor determinante para la disponibilidad de todo el entorno de SAP.

### <a name="sap-application-high-availability-on-azure-iaas"></a>Alta disponibilidad de las aplicaciones de SAP en IaaS de Azure

Para alcanzar una alta disponibilidad total en el sistema SAP, se deben proteger todos los componentes esenciales del sistema SAP como, por ejemplo, los servidores de aplicaciones de SAP redundantes y los componentes únicos (por ejemplo, único punto de error) como la instancia de SAP (A)SCS y el DBMS.

#### <a name="high-availability-for-sap-application-servers"></a><a name="5d9d36f9-9058-435d-8367-5ad05f00de77"></a>Alta disponibilidad en los servidores de aplicaciones de SAP

Para las instancias de diálogo o de servidores de aplicaciones de SAP no hay que buscar una solución específica de alta disponibilidad. La alta disponibilidad se consigue simplemente mediante la redundancia y, por tanto, disponiendo de la redundancia suficiente en las distintas máquinas virtuales. Todas se deben colocar en el mismo conjunto de disponibilidad de Azure para evitar la posibilidad de que las máquinas virtuales se actualicen al mismo tiempo durante el tiempo de inactividad planeado de mantenimiento. La funcionalidad básica, que se basa en diversos dominios de actualización y de error dentro de una unidad de escalado de Azure, ya se presentó en el capítulo [Dominios de actualización][planning-guide-3.2.2]. Los conjuntos de disponibilidad de Azure se presentaron en el capítulo [Conjuntos de disponibilidad de Azure][planning-guide-3.2.3] de este documento.

El número de dominios de error y de actualización que puede utilizar un conjunto de disponibilidad de Azure dentro de una unidad de escalado de Azure no es infinito. Esto significa que, al colocar un número de máquinas virtuales dentro de un conjunto de disponibilidad, tarde o temprano más de una máquina virtual acabará en el mismo dominio de error o de actualización.

Si se implementan unas cuantas instancias del servidor de aplicaciones de SAP en sus máquinas virtuales dedicadas y se parte de la base de que hay cinco dominios de actualización, al final se obtendrá la imagen siguiente. El número máximo real de dominios de error y de actualización dentro de un conjunto de disponibilidad podría cambiar en el futuro:

![Alta disponibilidad de los servidores de aplicaciones de SAP en Azure][planning-guide-figure-3000]

Se puede encontrar más información en esta documentación: <https://azure.microsoft.com/documentation/articles/virtual-machines-manage-availability>

#### <a name="high-availability-for-sap-central-services-on-azure"></a>Alta disponibilidad para los servicios centrales de SAP en Azure

Para información acerca de la arquitectura de alta disponibilidad de los servicios centrales de SAP en Azure, consulte el artículo [Escenarios y arquitectura de alta disponibilidad para SAP NetWeaver](./sap-high-availability-architecture-scenarios.md). El artículo contiene descripciones más detalladas para los sistemas operativos específicos.

#### <a name="high-availability-for-the-sap-database-instance"></a>Alta disponibilidad en la instancia de base de datos de SAP

La configuración típica de alta disponibilidad del DBMS de SAP se basa en dos máquinas virtuales de DBMS donde la funcionalidad de alta disponibilidad del DBMS se utiliza para replicar datos de la instancia activa del DBMS a la segunda máquina virtual en una instancia pasiva del DBMS.

La funcionalidad de recuperación ante desastres y de alta disponibilidad del DBMS en general, así como de los DBMS específicos, se describe en la [Guía de implementación de DBMS][dbms-guide].

#### <a name="end-to-end-high-availability-for-the-complete-sap-system"></a>Alta disponibilidad de un extremo a otro en todo el sistema SAP

A continuación, se ofrecen dos ejemplos de una arquitectura completa de alta disponibilidad de SAP NetWeaver en Azure: una para Windows y otra para Linux.

Solo discos no administrados: Es posible que se deba cuestionar la siguiente explicación de los conceptos cuando se implementen muchos sistemas SAP y que el número de máquinas virtuales implementadas supere el límite máximo de cuentas de almacenamiento por suscripción. En tales casos, se deben combinar los VHD de las máquinas virtuales dentro de una cuenta de almacenamiento. Normalmente, este procedimiento se llevaría a cabo combinando los VHD de las máquinas virtuales de la capa de aplicación de SAP en diversos sistemas SAP.  También se combinan varios VHD de diversas máquinas virtuales del DBMS de sistemas SAP distintos en una sola cuenta de Azure Storage. Por lo tanto, se tienen en cuenta los límites de IOPS de las cuentas de Azure Storage (<https://azure.microsoft.com/documentation/articles/storage-scalability-targets>)


##### <a name="windowslogo_windows-ha-on-windows"></a>![Windows][Logo_Windows] Alta disponibilidad en Windows

![Arquitectura de alta disponibilidad de las aplicaciones de SAP NetWeaver con SQL Server en IaaS de Azure][planning-guide-figure-3200]

Las siguientes construcciones de Azure se utilizan en el sistema SAP NetWeaver para minimizar el impacto causado por problemas de infraestructura y la aplicación de revisiones en el host:

* El sistema completo se implementa en Azure (obligatorio: la capa del DBMS, la instancia de [A]SCS y la capa de aplicación completa deben ejecutarse en la misma ubicación).
* El sistema se ejecuta completamente dentro de una suscripción de Azure (obligatorio).
* El sistema se ejecuta completamente dentro de una instancia de Azure Virtual Network (obligatorio).
* Las máquinas virtuales de un sistema SAP se pueden dividir en tres conjuntos de disponibilidad aunque todas las máquinas virtuales pertenezcan a la misma red virtual.
* Cada capa (por ejemplo, DBMS, ASCS, servidores de aplicaciones) tiene que usar un conjunto de disponibilidad dedicado.
* Todas las máquinas virtuales que ejecutan instancias del DBMS de un sistema SAP se encuentran en un único conjunto de disponibilidad. Se presupone que hay más de una máquina virtual que ejecuta instancias del DBMS por sistema, ya que se utilizan características nativas de alta disponibilidad del DBMS como SQL Server AlwaysOn u Oracle Data Guard.
* Todas las máquinas virtuales que ejecutan instancias del DBMS utilizan su propia cuenta de almacenamiento. Los archivos de registro y datos del DBMS se replican desde una cuenta de almacenamiento a otra mediante funciones de alta disponibilidad del DBMS que sincronizan los datos. La indisponibilidad de una cuenta de almacenamiento ocasionará que uno de los nodos de clúster de SQL de Windows deje de estar disponible, pero no el servicio de SQL Server al completo.
* Todas las máquinas virtuales que ejecutan la instancia de (A)SCS de un sistema SAP se encuentran en un único conjunto de disponibilidad. Un clúster de conmutación por error de Windows Server (WSFC) se configura dentro de las máquinas virtuales para proteger la instancia de (A)SCS.
* Todas las máquinas virtuales que ejecutan instancias de (A)SCS utilizan su propia cuenta de almacenamiento. Los archivos de la instancia de (A)SCS y la carpeta global de SAP se replican de una cuenta de almacenamiento a otra mediante la replicación de SIOS DataKeeper. La indisponibilidad de una cuenta de almacenamiento ocasionará que uno de los nodos de clúster de (A)SCS de Windows deje de estar disponible, pero no el servicio de (A)SCS al completo.
* TODAS las máquinas virtuales que representa la capa del servidor de aplicaciones de SAP se encuentran en un tercer conjunto de disponibilidad.
* TODAS las máquinas virtuales que ejecutan servidores de aplicaciones de SAP usan su propia cuenta de almacenamiento. La indisponibilidad de una cuenta de almacenamiento ocasionará que un servidor de aplicaciones de SAP deje de estar disponible, mientras que el resto de los servidores de aplicaciones de SAP continuarán ejecutándose.

En la siguiente figura se ilustra el mismo entorno con la utilización de discos de Managed Disks.

![Arquitectura de alta disponibilidad de las aplicaciones de SAP NetWeaver con SQL Server en IaaS de Azure][planning-guide-figure-3201]

##### <a name="linuxlogo_linux-ha-on-linux"></a>![Linux][Logo_Linux] Alta disponibilidad en Linux

La arquitectura de alta disponibilidad de SAP en Linux en Azure es básicamente la misma que la descrita anteriormente para Windows. Consulte la nota de SAP [1928533] para obtener una lista de soluciones compatibles y de alta disponibilidad.

### <a name="using-autostart-for-sap-instances"></a><a name="4e165b58-74ca-474f-a7f4-5e695a93204f"></a>Uso de Autostart en las instancias de SAP

SAP ofrecía una funcionalidad para iniciar las instancias de SAP inmediatamente después de iniciar el sistema operativo en la máquina virtual. Los pasos exactos están documentados en el artículo de Knowledge Base sobre SAP [1909114]. Sin embargo, SAP recomienda dejar de utilizar esta configuración, ya que no hay ningún control del orden en el que se reinicia la instancia en el caso de que se vean afectadas más de una máquina virtual o que se ejecuten varias instancias por máquina virtual. Si se presupone un escenario de Azure típico de una instancia del servidor de aplicaciones de SAP en una máquina virtual y la circunstancia de que acabe reiniciándose una sola máquina virtual, Autostart no planteará ningún verdadero problema, y podrá habilitarse agregando este parámetro:

`Autostart = 1`

Se debe agregar en el perfil de inicio de la instancia de ABAP o Java de SAP.

> [!NOTE]
> El parámetro Autostart también puede presentar algunas deficiencias. Concretamente, este parámetro desencadena el inicio de una instancia de Java o ABAP de SAP cuando se inicia el servicio de Windows/Linux relacionado de la instancia. No cabe duda de que esto es lo que sucede cuando se inicia el sistema operativo. Sin embargo, los reinicios de los servicios de SAP también son frecuentes en la funcionalidad de administración del ciclo de vida de software de SAP como SUM u otras actualizaciones o modificaciones. Estas funcionalidades no esperan en absoluto que una instancia se reinicie automáticamente. Por lo tanto, se debe deshabilitar el parámetro Autostart antes de ejecutar tales tareas. El parámetro Autostart tampoco debe usarse en las instancias de SAP agrupadas en clústeres como ASCS, SCS o CI.
>
>

Se puede consultar más información sobre el inicio automático de las instancias de SAP aquí:

* [Start/Stop SAP along with your Unix Server Start/Stop (Inicio/detención de SAP al iniciar/detener el servidor de Unix)](https://scn.sap.com/community/unix/blog/2012/08/07/startstop-sap-along-with-your-unix-server-startstop)
* [Starting and Stopping SAP NetWeaver Management Agents (Inicio y detención de los agentes de administración de SAP NetWeaver)](https://help.sap.com/saphelp_nwpi711/helpdata/en/49/9a15525b20423ee10000000a421938/content.htm)
* [How to enable auto Start of HANA Database (Habilitación del inicio automático de la base de datos HANA)](http://sapbasisinfo.com/blog/2016/08/15/enabling-autostart-of-sap-hana-database-on-server-boot-situation/)

### <a name="larger-3-tier-sap-systems"></a>Sistemas SAP de tres niveles más grandes
Los aspectos relativos a la alta disponibilidad de las configuraciones de SAP de tres niveles ya se comentaron en las secciones anteriores. ¿Pero qué sucede con los sistemas en los que los requisitos del servidor del DBMS son demasiado grandes como para que este se encuentre en Azure, pero la capa de aplicación de SAP podría implementarse en Azure?

#### <a name="location-of-3-tier-sap-configurations"></a>Ubicación de las configuraciones de SAP de tres niveles
No se puede dividir la propia capa de aplicación ni la aplicación y la capa del DBMS entre un sistema local y Azure. Un sistema SAP se debe implementar completamente en el entorno local O en Azure. Tampoco se puede hacer que algunos de los servidores de aplicaciones se ejecuten en el entorno local y otros, en Azure. Este es el punto de partida de esta exposición. Tampoco se permite implementar los componentes del DBMS de un sistema SAP y la capa del servidor de aplicaciones de SAP en dos regiones de Azure diferentes. Por ejemplo, implementar DBMS en la región Oeste de EE. UU. y la capa de la aplicación de SAP en Centro de EE. UU. El motivo para no admitir tales configuraciones es la sensibilidad a la latencia de la arquitectura de SAP NetWeaver.

Sin embargo, a lo largo del año pasado, unos asociados de centro de datos desarrollaron coubicaciones de las regiones de Azure. Dichas coubicaciones suelen estar muy cerca de los centros de datos físicos de Azure de una región de Azure. Esta reducida distancia y la conexión de los activos de la coubicación con Azure por medio de ExpressRoute pueden generar una latencia inferior a 2 ms. En tales casos, se puede localizar la capa del DBMS (incluidos la SAN o el NAS de almacenamiento) en esta coubicación y la capa de aplicación de SAP en Azure. [Instancias grandes de HANA](./hana-overview-architecture.md). 

### <a name="offline-backup-of-sap-systems"></a>Copia de seguridad sin conexión de sistemas SAP
En función de la configuración de SAP elegida (de dos o de tres niveles) es posible que se deba realizar una copia de seguridad, tanto del contenido de la propia máquina virtual como de la base de datos. Se espera que las copias de seguridad relacionadas con el DBMS se realicen mediante métodos de base de datos. Se puede encontrar una descripción detallada de las distintas bases de datos en la [Guía de implementación de DBMS][dbms-guide]. Por otro lado, se puede realizar una copia de seguridad sin conexión de los datos de SAP (incluido el contenido de la base de datos) tal y como se describe en esta sección o realizarla en línea como se describe en la sección siguiente.

La copia de seguridad sin conexión requiere básicamente apagar la máquina virtual mediante Azure Portal, así como realizar una copia del disco de la VM base y de todos los discos conectados a la máquina virtual. De este modo, se puede conservar una imagen de la VM y su disco asociado en un momento dado. Se recomienda copiar las copias de seguridad en otra cuenta de Azure Storage. Por lo tanto, se aplicaría el procedimiento descrito en el capítulo [Copia de discos entre cuentas de Azure Storage][planning-guide-5.4.2] de este documento.
Además del apagado mediante Azure Portal, también se puede realizar mediante PowerShell o la CLI, como se describe aquí: <https://azure.microsoft.com/documentation/articles/virtual-machines-deploy-rmtemplates-powershell/>

Para realizar una restauración a este estado, se debe eliminar la VM base, así como los discos originales y los discos montados de dicha VM; volver a copiar los discos guardados en la cuenta de almacenamiento original o el grupo de recursos de los discos de Managed Disks y, luego, volver a implementar el sistema.
En este artículo se muestra un ejemplo de cómo crear un script para este proceso en Powershell: <http://www.westerndevs.com/azure-snapshots/>.

Hay que asegurarse de instalar una nueva licencia de SAP, ya que, al restaurar una copia de seguridad de la VM del modo descrito más arriba, se crea una clave de hardware.

### <a name="online-backup-of-an-sap-system"></a>Copia de seguridad en línea de un sistema SAP

Las copias de seguridad de DBMS se realizan mediante métodos específicos para DBMS descritos en la [Guía de DBMS][dbms-guide].

Se puede efectuar una copa de seguridad de otras máquinas virtuales dentro del sistema SAP mediante la funcionalidad de copia de seguridad de Azure Virtual Machines. Esta funcionalidad es un método estándar para la realización de una copia de seguridad de una máquina virtual completa en Azure. Azure Backup almacena las copias de seguridad en Azure y permite volver a restaurarlas.

> [!NOTE]
> Desde diciembre de 2015, la copia de seguridad de máquinas virtuales NO conserva el identificador único de VM que se utiliza para la concesión de licencias de SAP. Esto significa que restaurar la copia de seguridad de una máquina virtual requiere instalar una clave de licencia de SAP nueva, ya que se considera que la máquina virtual restaurada es una máquina virtual nueva en lugar de una sustitución de la anterior que estaba guardada.
>
> ![Windows][Logo_Windows] Windows
>
> En teoría, se puede realizar una copia de seguridad de las máquinas virtuales que ejecutan bases de datos de forma coherente si el sistema de DBMS admite Windows VSS (Servicio de instantáneas de volumen de Windows <https://msdn.microsoft.com/library/windows/desktop/bb968832(v=vs.85).aspx>) como hace SQL Server, por ejemplo.
> Sin embargo, se debe tener en cuenta que no se pueden restaurar bases de datos en un momento dado a partir de las copias de seguridad de máquinas virtuales de Azure. Por lo tanto, se recomienda realizar copias de seguridad de las bases de datos con la funcionalidad del DBMS en lugar de confiar en la copia de seguridad de máquinas virtuales de Azure.
>
> Para familiarizarse con la copia de seguridad de máquinas virtuales de Azure, empiece aquí: <https://docs.microsoft.com/azure/backup/backup-azure-vms>.
>
> Otras posibilidades son usar una combinación del Microsoft Data Protection Manager instalado en una máquina virtual de Azure y Azure Backup para efectuar copias de seguridad o restaurar bases de datos. Se puede encontrar más información aquí: <https://docs.microsoft.com/azure/backup/backup-azure-dpm-introduction>.  
>
> ![Linux][Logo_Linux] Linux
>
> No hay ningún equivalente de Windows VSS en Linux. Por lo tanto, solo se pueden realizar copias de seguridad coherentes con archivos, pero no copias de seguridad coherentes con la aplicación. La copia de seguridad del DBMS de SAP se debe efectuar mediante la funcionalidad del DBMS. El sistema de archivos que incluye los datos relacionados con SAP se puede guardar, por ejemplo, mediante tar, como se describe aquí: <https://help.sap.com/saphelp_nw70ehp2/helpdata/en/d3/c0da3ccbb04d35b186041ba6ac301f/content.htm>.
>
>

### <a name="azure-as-dr-site-for-production-sap-landscapes"></a>Azure como sitio de recuperación ante desastres para entornos de producción de SAP

Desde mediados de 2014, las extensiones de diversos componentes de Hyper-V, System Center y Azure permiten el uso de Azure como sitio de recuperación ante desastres para las máquinas virtuales basadas en Hyper-V que se ejecutan en el entorno local.

Se puede encontrar un blog que detalla cómo implementar esta solución aquí: <https://blogs.msdn.com/b/saponsqlserver/archive/2014/11/19/protecting-sap-solutions-with-azure-site-recovery.aspx>.

## <a name="summary"></a>Resumen

Los aspectos más destacados de la alta disponibilidad en los sistemas SAP en Azure son los siguientes:

* En estos momentos, el único punto de error de SAP no se puede proteger exactamente del mismo modo que en las implementaciones locales. El motivo es que todavía no se pueden compilar clústeres de disco compartido en Azure sin recurrir a software de terceros.
* Para la capa del DBMS, se debe utilizar una funcionalidad del DBMS que no se basa en la tecnología de clúster de disco compartido. Los detalles están documentados en la [Guía de DBMS][dbms-guide].
* Para minimizar el impacto de los problemas que se produzcan en los dominios de error sobre la infraestructura de Azure o el mantenimiento de hosts, se deben usar los conjuntos de disponibilidad de Azure:
  * Se recomienda tener un conjunto de disponibilidad para la capa de aplicación de SAP.
  * Se recomienda tener un conjunto de disponibilidad independiente para la capa del DBMS de SAP.
  * NO se recomienda aplicar el mismo conjunto de disponibilidad a las máquinas virtuales de sistemas SAP distintos.
  * Se recomienda utilizar Premium Managed Disks.
* Para realizar copias de seguridad de la capa de DBMS de SAP, consulte la [Guía de DBMS][dbms-guide].
* No tiene mucho sentido efectuar una copia de seguridad de las instancias de diálogo de SAP, ya que volver a implementar instancias de diálogo sencillas suele ser un método más rápido.
* Por el contrario, sí que tiene sentido realizar una copia de seguridad de la máquina virtual que contiene el directorio global del sistema SAP y, con él, todos los perfiles de las distintas instancias, lo cual debe hacerse mediante las copias de seguridad de Windows o, por ejemplo, mediante tar en Linux. Dado que existen diferencias entre Windows Server 2008 (R2) y Windows Server 2012 (R2), pues las últimas versiones de Windows Server facilitan la realización de copias de seguridad, se recomienda ejecutar Windows Server 2012 (R2) como sistema operativo invitado Windows.

## <a name="next-steps"></a>Pasos siguientes
Lea los artículos:

- [Implementación de Azure Virtual Machines para SAP NetWeaver](./deployment-guide.md)
- [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](./dbms_guide_general.md)
- [Configuraciones y operaciones de infraestructura de SAP HANA en Azure](/- azure/virtual-machines/workloads/sap/hana-vm-operations)
