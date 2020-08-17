---
title: Implementación de SAP MaxDB, liveCache y del servidor de contenido en máquinas virtuales de Azure | Microsoft Docs
description: Implementación de SAP MaxDB, liveCache y del servidor de contenido en Azure
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/12/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 842ab7a1562c731e790ba03b2fd5acdc3987a90d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87051955"
---
# <a name="sap-maxdb-livecache-and-content-server-deployment-on-azure-vms"></a>Implementación de SAP MaxDB, liveCache y del servidor de contenido en máquinas virtuales de Azure

[767598]:https://launchpad.support.sap.com/#/notes/767598
[773830]:https://launchpad.support.sap.com/#/notes/773830
[826037]:https://launchpad.support.sap.com/#/notes/826037
[965908]:https://launchpad.support.sap.com/#/notes/965908
[1031096]:https://launchpad.support.sap.com/#/notes/1031096
[1114181]:https://launchpad.support.sap.com/#/notes/1114181
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
[2171857]:https://launchpad.support.sap.com/#/notes/2171857
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
[dbms-guide-managed-disks]:dbms-guide.md#f42c6cb5-d563-484d-9667-b07ae51bce29

[dbms-guide-figure-100]:media/virtual-machines-shared-sap-dbms-guide/100_storage_account_types.png
[dbms-guide-figure-200]:media/virtual-machines-shared-sap-dbms-guide/200-ha-set-for-dbms-ha.png
[dbms-guide-figure-300]:media/virtual-machines-shared-sap-dbms-guide/300-reference-config-iaas.png
[dbms-guide-figure-400]:media/virtual-machines-shared-sap-dbms-guide/400-sql-2012-backup-to-blob-storage.png
[dbms-guide-figure-500]:media/virtual-machines-shared-sap-dbms-guide/500-sql-2012-backup-to-blob-storage-different-containers.png
[dbms-guide-figure-600]:media/virtual-machines-shared-sap-dbms-guide/600-iaas-maxdb.png
[dbms-guide-figure-700]:media/virtual-machines-shared-sap-dbms-guide/700-livecach-prod.png
[dbms-guide-figure-800]:media/virtual-machines-shared-sap-dbms-guide/800-azure-vm-sap-content-server.png
[dbms-guide-figure-900]:media/dbms_maxdb_deployment_guide/900-sap-cache-server-on-premises.png

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

[msdn-set-azurermvmaemextension]:https://msdn.microsoft.com/library/azure/mt670598.aspx

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
[planning-guide-figure-2500]:media/virtual-machines-shared-sap-planning-guide/2500-vm-extension-details.png
[planning-guide-figure-2600]:media/virtual-machines-shared-sap-planning-guide/2600-sap-router-connection.png
[planning-guide-figure-2700]:media/virtual-machines-shared-sap-planning-guide/2700-exposed-sap-portal.png
[planning-guide-figure-2800]:media/virtual-machines-shared-sap-planning-guide/2800-endpoint-config.png
[planning-guide-figure-2900]:media/virtual-machines-shared-sap-planning-guide/2900-azure-ha-sap-ha.png
[planning-guide-figure-300]:media/virtual-machines-shared-sap-planning-guide/300-vpn-s2s.png
[planning-guide-figure-3000]:media/virtual-machines-shared-sap-planning-guide/3000-sap-ha-on-azure.png
[planning-guide-figure-3200]:media/virtual-machines-shared-sap-planning-guide/3200-sap-ha-with-sql.png
[planning-guide-figure-400]:media/virtual-machines-shared-sap-planning-guide/400-vm-services.png
[planning-guide-figure-600]:media/virtual-machines-shared-sap-planning-guide/600-s2s-details.png
[planning-guide-figure-700]:media/virtual-machines-shared-sap-planning-guide/700-decision-tree-deploy-to-azure.png
[planning-guide-figure-800]:media/virtual-machines-shared-sap-planning-guide/800-portal-vm-overview.png
[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd 
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f 

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
[storage-powershell-guide-full-copy-vhd]:../../../storage/common/storage-powershell-guide-full.md#how-to-copy-blobs-from-one-storage-container-to-another
[storage-premium-storage-preview-portal]:../../windows/disks-types.md
[storage-redundancy]:../../../storage/common/storage-redundancy.md
[storage-scalability-targets]:../../../storage/common/scalability-targets-standard-accounts.md
[storage-use-azcopy]:../../../storage/common/storage-use-azcopy.md
[template-201-vm-from-specialized-vhd]:https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-from-specialized-vhd
[templates-101-simple-windows-vm]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-simple-windows-vm
[templates-101-vm-from-user-image]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image
[virtual-machines-linux-attach-disk-portal]:../../linux/attach-disk-portal.md
[virtual-machines-azure-resource-manager-architecture]:../../../resource-manager-deployment-model.md
[virtual-machines-azurerm-versus-azuresm]:../../../resource-manager-deployment-model.md
[virtual-machines-windows-classic-configure-oracle-data-guard]:../../virtual-machines-windows-classic-configure-oracle-data-guard.md
[virtual-machines-linux-cli-deploy-templates]:../../linux/cli-deploy-templates.md 
[virtual-machines-deploy-rmtemplates-powershell]:../../virtual-machines-windows-ps-manage.md 
[virtual-machines-linux-agent-user-guide]:../../linux/agent-user-guide.md
[virtual-machines-linux-agent-user-guide-command-line-options]:../../linux/agent-user-guide.md#command-line-options
[virtual-machines-linux-capture-image]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager-capture]:../../linux/capture-image.md#step-2-capture-the-vm
[virtual-machines-linux-configure-raid]:../../linux/configure-raid.md
[virtual-machines-linux-configure-lvm]:../../linux/configure-lvm.md
[virtual-machines-linux-classic-create-upload-vhd-step-1]:../../virtual-machines-linux-classic-create-upload-vhd.md#step-1-prepare-the-image-to-be-uploaded
[virtual-machines-linux-create-upload-vhd-suse]:../../linux/suse-create-upload-vhd.md
[virtual-machines-linux-redhat-create-upload-vhd]:../../linux/redhat-create-upload-vhd.md
[virtual-machines-linux-how-to-attach-disk]:../../linux/add-disk.md
[virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux]:../../linux/add-disk.md#connect-to-the-linux-vm-to-mount-the-new-disk
[virtual-machines-linux-tutorial]:../../linux/quick-create-cli.md
[virtual-machines-linux-update-agent]:../../linux/update-agent.md
[virtual-machines-manage-availability-linux]:../../linux/manage-availability.md
[virtual-machines-manage-availability-windows]:../../windows/manage-availability.md
[virtual-machines-ps-create-preconfigure-windows-resource-manager-vms]:virtual-machines-windows-create-powershell.md
[virtual-machines-sizes-linux]:../../linux/sizes.md
[virtual-machines-sizes-windows]:../../windows/sizes.md
[virtual-machines-windows-classic-ps-sql-alwayson-availability-groups]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups.md
[virtual-machines-windows-classic-ps-sql-int-listener]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener.md
[virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions]:../../../azure-sql/virtual-machines/windows/business-continuity-high-availability-disaster-recovery-hadr-overview.md
[virtual-machines-sql-server-infrastructure-services]:../../../azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md
[virtual-machines-sql-server-performance-best-practices]:../../../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md
[virtual-machines-upload-image-windows-resource-manager]:../../virtual-machines-windows-upload-image.md
[virtual-machines-windows-tutorial]:../../virtual-machines-windows-hero-tutorial.md
[virtual-machines-workload-template-sql-alwayson]:https://azure.microsoft.com/resources/templates/sql-server-2014-alwayson-existing-vnet-and-ad/
[virtual-network-deploy-multinic-arm-cli]:../linux/multiple-nics.md
[virtual-network-deploy-multinic-arm-ps]:../windows/multiple-nics.md
[virtual-network-deploy-multinic-arm-template]:../../../virtual-network/template-samples.md
[virtual-networks-configure-vnet-to-vnet-connection]:../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md
[virtual-networks-create-vnet-arm-pportal]:../../../virtual-network/manage-virtual-network.md#create-a-virtual-network
[virtual-networks-manage-dns-in-vnet]:../../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md
[virtual-networks-multiple-nics]:../../../virtual-network/virtual-network-deploy-multinic-classic-ps.md
[virtual-networks-nsg]:../../../virtual-network/security-overview.md
[virtual-networks-reserved-private-ip]:../../../virtual-network/virtual-networks-static-private-ip-arm-ps.md
[virtual-networks-static-private-ip-arm-pportal]:../../../virtual-network/virtual-networks-static-private-ip-arm-pportal.md
[virtual-networks-udr-overview]:../../../virtual-network/virtual-networks-udr-overview.md
[vpn-gateway-about-vpn-devices]:../../../vpn-gateway/vpn-gateway-about-vpn-devices.md
[vpn-gateway-create-site-to-site-rm-powershell]:../../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md
[vpn-gateway-cross-premises-options]:../../../vpn-gateway/vpn-gateway-plan-design.md
[vpn-gateway-site-to-site-create]:../../../vpn-gateway/vpn-gateway-site-to-site-create.md
[vpn-gateway-vpn-faq]:../../../vpn-gateway/vpn-gateway-vpn-faq.md
[xplat-cli]:../../../cli-install-nodejs.md
[xplat-cli-azure-resource-manager]:../../../xplat-cli-azure-resource-manager.md




En este documento se describen las diferentes áreas que se deben tener en cuenta al implementar MaxDB, liveCache y el servidor de contenido en IaaS de Azure. Como condición previa a este documento, debe haber leído el documento [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md), así como otras guías de la [documentación de carga de trabajo de SAP en Azure](./get-started.md). 

## <a name="specifics-for-the-sap-maxdb-deployments-on-windows"></a>Detalles para las implementaciones de SAP MaxDB en Windows
### <a name="sap-maxdb-version-support-on-azure"></a>Compatibilidad de versiones de SAP MaxDB en Azure
En la actualidad, SAP admite la versión 7.9 de SAP MaxDB o superior para su uso con productos basados en SAP NetWeaver en Azure. Todas las actualizaciones del servidor SAP MaxDB o de los controladores ODBC y JDBC que se utilizarán con productos basados en SAP NetWeaver se proporcionan únicamente a través de SAP Service Marketplace en <https://support.sap.com/swdc>.
Se puede encontrar información general sobre la ejecución de SAP NetWeaver en SAP MaxDB en <https://www.sap.com/community/topic/maxdb.html>.

### <a name="supported-microsoft-windows-versions-and-azure-vm-types-for-sap-maxdb-dbms"></a>Versiones de Microsoft Windows y tipos de máquina virtual de Azure admitidos para SAP MaxDB DBMS
Para obtener la versión compatible de Microsoft Windows para SAP MaxDB DBMS en Azure, consulte:

* [Matriz de disponibilidad de productos (PAM) SAP][sap-pam]
* Nota de SAP [1928533]

Se recomienda encarecidamente usar la versión más reciente del sistema operativo Microsoft Windows (Microsoft Windows 2016).

### <a name="available-sap-maxdb-documentation-for-maxdb"></a>Documentación de SAP MaxDB disponible para MaxDB
Encontrará la lista de la documentación de SAP MaxDB actualizada en la nota de SAP [767598]

### <a name="sap-maxdb-configuration-guidelines-for-sap-installations-in-azure-vms"></a>Instrucciones de configuración de SAP MaxDB para instalaciones de SAP en máquinas virtuales de Azure
#### <a name="storage-configuration"></a><a name="b48cfe3b-48e9-4f5b-a783-1d29155bd573"></a>Configuración de almacenamiento
Los procedimientos recomendados de almacenamiento de Azure para SAP MaxDB siguen las recomendaciones generales mencionadas en el capítulo [Estructura de almacenamiento de una máquina virtual para implementaciones de RDBMS](./dbms_guide_general.md#65fa79d6-a85f-47ee-890b-22e794f51a64).

> [!IMPORTANT]
> Al igual que otras bases de datos, SAP MaxDB también dispone de archivos de datos y de registro. Sin embargo, en la terminología de SAP MaxDB el término correcto es "volumen" (no "archivo"). Por ejemplo, existen volúmenes de datos y de registro de SAP MaxDB. No los confunda con los volúmenes de disco del sistema operativo. 
> 
> 

En resumen, deberá hacer lo siguiente:

* Si usa cuentas de Azure Storage, establezca la cuenta que contiene los volúmenes de registro y datos de SAP MaxDB (los archivos de datos y registros) en **Almacenamiento con redundancia local (LRS)** como se indica en el capítulo [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md).
* Separe la ruta de acceso de E/S para los volúmenes de datos de SAP MaxDB (los archivos de datos) de la ruta de acceso de E/S para los volúmenes de registro (los archivos de registro). Esto significa que los volúmenes de datos de SAP MaxDB (los archivos de datos) se tienen que instalar en una unidad lógica y los volúmenes de registro de SAP MaxDB (los archivos de registro) en otra.
* Establezca el tipo de almacenamiento en caché adecuado para cada disco en función de si lo usa para volúmenes de datos o registros de SAP MaxDB (archivos de datos o de registro) y de si se usa Azure Standard Storage o Azure Premium Storage, como se describe en [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md).
* Mientras la cuota actual de IOPS por disco cumpla los requisitos, se pueden almacenar todos los volúmenes de datos en un solo disco montado y también almacenar todos los volúmenes de registro de bases de datos en otro único disco montado.
* Si se necesita más IOPS o espacio, se recomienda usar bloques de almacenamiento de Microsoft Windows (solo disponibles en Microsoft Windows Server 2012 y versiones posteriores) para crear un dispositivo lógico de gran tamaño en varios discos montados. Para obtener más información, vea [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md). Este enfoque simplifica la sobrecarga de administración para administrar el espacio en disco y evita el esfuerzo de distribuir archivos manualmente en varios discos montados.
* Se recomienda usar Azure Premium Storage para las implementaciones de MaxDB. 

![Configuración de referencia de máquinas virtuales IaaS de Azure para SAP MaxDB DBMS](./media/dbms_maxdb_deployment_guide/Simple_disk_structure_maxdb.PNG)


#### <a name="backup-and-restore"></a><a name="23c78d3b-ca5a-4e72-8a24-645d141a3f5d"></a>Copia de seguridad y restauración
Al implementar SAP MaxDB en Azure, debe revisar la metodología de copias de seguridad. Incluso si el sistema no es productivo, se debe realizar periódicamente la copia de seguridad de la base de datos de SAP que hospeda SAP MaxDB. Puesto que Azure Storage guarda tres imágenes, ahora una copia de seguridad tiene menos relevancia en cuanto a la protección del sistema contra errores de almacenamiento y más relevancia en cuanto a errores operativos o administrativos. La razón principal para el mantenimiento de un plan de copia de seguridad y restauración correcto es que puede compensar errores lógicos o manuales gracias a la capacidad de recuperación a un momento dado. Por tanto, el propósito es usar copias de seguridad para restaurar la base de datos a un momento dado o utilizar las copias de seguridad de Azure para propagar otro sistema mediante la copia de la base de datos existente. 

La realización de copias de seguridad y restauraciones de una base de datos de Azure funciona del mismo modo que para sistemas locales, por lo que puede usar herramientas estándar de copia de seguridad y restauración de SAP MaxDB, que se describen en uno de los documentos de SAP MaxDB enumerados en la nota de SAP [767598]. 

#### <a name="performance-considerations-for-backup-and-restore"></a><a name="77cd2fbb-307e-4cbf-a65f-745553f72d2c"></a>Consideraciones de rendimiento para copias de seguridad y restauraciones
Como en las implementaciones sin sistema operativo, el rendimiento de las copias de seguridad y las restauraciones depende de cuántos volúmenes se puedan leer en paralelo y del rendimiento de estos volúmenes. Por lo tanto, se pueden asumir los siguientes puntos:

* Cuantos menos discos se utilicen para almacenar los dispositivos de las bases de datos, menor será el rendimiento general de lectura
* Cuantos menos destinos (directorios de seccionamiento, discos) en los que escribir la copia de seguridad haya, menor será el rendimiento

Para aumentar la cantidad de destinos en los que escribir, existen dos opciones a las que puede recurrir (puede que de forma combinada) según sus necesidades:

* Dedicar volúmenes distintos a copias de seguridad
* Seccionar el volumen de destino de la copia de seguridad en varios discos montados con el fin de mejorar el rendimiento de IOPS en ese volumen de disco seccionado
* Disponer de dispositivos de discos lógicos dedicados distintos para:
  * Volúmenes (archivos) de copia de seguridad de SAP MaxDB
  * Volúmenes de datos de SAP MaxDB (archivos)
  * Volúmenes de registro de SAP MaxDB (archivos)

El particionamiento de un volumen en varios discos montados se ha explicado anteriormente en [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md). 

#### <a name="other-considerations"></a><a name="f77c1436-9ad8-44fb-a331-8671342de818"></a>Otras consideraciones
También se aplican todas las demás áreas generales, como los conjuntos de disponibilidad de Azure o la supervisión de SAP, como se describe en [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md)  a las implementaciones de máquinas virtuales con la base de datos de SAP MaxDB.
Otras configuraciones específicas de SAP MaxDB son transparentes para las máquinas virtuales de Azure y se describen en distintos documentos de la nota de SAP [767598] y en estas otras:

* [826037] 
* [1139904]
* [1173395]

## <a name="specifics-for-sap-livecache-deployments-on-windows"></a>Detalles para implementaciones de SAP liveCache en Windows
### <a name="sap-livecache-version-support"></a>Compatibilidad de versiones de SAP liveCache
La versión mínima de SAP liveCache que se admite en Azure Virtual Machines es **SAP LC/LCAPPS 10.0 SP 25**, incluida **liveCache 7.9.08.31** y **LCA-Build 25**, publicadas para **EhP 2 para SAP SCM 7.0** y versiones posteriores.

### <a name="supported-microsoft-windows-versions-and-azure-vm-types-for-sap-livecache-dbms"></a>Versiones de Microsoft Windows y tipos de máquina virtual de Azure admitidos para SAP liveCache DBMS
Para obtener la versión compatible de Microsoft Windows para SAP liveCache en Azure, consulte:

* [Matriz de disponibilidad de productos (PAM) SAP][sap-pam]
* Nota de SAP [1928533]

Se recomienda encarecidamente utilizar la versión más reciente del sistema operativo Microsoft Windows Server. 

### <a name="sap-livecache-configuration-guidelines-for-sap-installations-in-azure-vms"></a>Instrucciones de configuración de SAP liveCache para instalaciones de SAP en máquinas virtuales de Azure
#### <a name="recommended-azure-vm-types-for-livecache"></a>Tipos de máquina virtual de Azure recomendadas para liveCache
Puesto que SAP liveCache es una aplicación que realiza cálculos de grandes proporciones, la cantidad y la velocidad de CPU y RAM influyen en gran medida en el rendimiento de SAP liveCache. 

En el caso de los tipos de máquina virtual de Azure compatibles con SAP (nota de SAP [1928533]), todos los recursos de CPU virtuales asignados a la máquina virtual cuentan con el respaldo de los recursos de CPU físicos dedicados del hipervisor. No existe aprovisionamiento en exceso (y, por tanto, tampoco existe competencia por los recursos de CPU).

Del mismo modo, para todos los tipos de instancia de máquina virtual de Azure compatibles con SAP, la memoria de la máquina virtual se asigna al 100 % a la memoria física; por ejemplo, no se recurre al aprovisionamiento en exceso (exceso de compromiso).

Desde esta perspectiva, es muy recomendable usar las máquinas virtuales Dv2, Dv3, Ev3 y de la serie M más recientes. La elección de los diferentes tipos de máquina virtual depende de la memoria que se necesita para liveCache y los recursos de CPU necesarios. Como sucede con todas las demás implementaciones de DBMS, es aconsejable aprovechar Azure Premium Storage para los volúmenes en los que el rendimiento es crucial.

#### <a name="storage-configuration-for-livecache-in-azure"></a>Configuración de almacenamiento para liveCache en Azure
Puesto que SAP liveCache se basa en la tecnología SAP MaxDB, todos los procedimientos recomendados de Azure Storage que se mencionan para SAP MaxDB en este documento también se pueden aplicar a SAP liveCache. 

#### <a name="dedicated-azure-vm-for-livecache-scenario"></a>Máquina virtual de Azure dedicada para el escenario de liveCache
Dado que SAP liveCache emplea una potencia de cálculo sumamente intensiva, se recomienda encarecidamente implementarla en una máquina virtual de Azure dedicada para obtener resultados productivos. 

![Máquina virtual de Azure dedicada para liveCache para un caso de uso productivo](./media/dbms_maxdb_deployment_guide/700-livecach-prod.PNG)


#### <a name="backup-and-restore-for-livecache-in-azure"></a>Copia de seguridad y restauración para liveCache en Azure
Las consideraciones sobre copia de seguridad y restauración ya se describen en los capítulos pertinentes de SAP MaxDB de este documento. 

#### <a name="other-considerations"></a>Otras consideraciones
Todas las demás áreas generales ya se describen en el capítulo de SAP MaxDB correspondiente. 

## <a name="specifics-for-the-sap-content-server-deployment-on-windows-in-azure"></a>Detalles para la implementación de SAP Content Server en Windows en Azure
El servidor SAP Content Server es un componente independiente basado en servidor para almacenar contenido como documentos electrónicos en formatos diferentes. SAP Content Server se ofrece para el desarrollo de tecnología y se concibe para su uso en varias aplicaciones con cualquier aplicación de SAP. Se instala en un sistema independiente. Su contenido más habitual es material de aprendizaje y documentación de dibujos técnicos de Knowledge Warehouse que se crean en el sistema de administración de documentos de mySAP PLM. 

### <a name="sap-content-server-version-support-for-azure-vms"></a>Compatibilidad de versiones de SAP Content Server para máquinas virtuales de Azure
SAP actualmente admite:

* **SAP Content Server** con la versión **6.50 (y superior)**
* **SAP MaxDB versión 7.9**
* **Microsoft IIS (Internet Information Server) versión 8.0 (y versiones posterior)**

Se recomienda encarecidamente usar las versiones más recientes de SAP Content Server y de **Microsoft IIS**. 

Compruebe las últimas versiones compatibles de SAP Content Server y Microsoft IIS en [Matriz de disponibilidad de productos (PAM) SAP][sap-pam].

### <a name="supported-microsoft-windows-and-azure-vm-types-for-sap-content-server"></a>Tipos de máquina virtual de Azure y Microsoft Windows admitidos con SAP Content Server
Para conocer la versión de Windows compatible con SAP Content Server en Azure, consulte:

* [Matriz de disponibilidad de productos (PAM) SAP][sap-pam]
* Nota de SAP [1928533]

Se recomienda encarecidamente utilizar la versión más reciente de Microsoft Windows Server.

### <a name="sap-content-server-configuration-guidelines-for-sap-installations-in-azure-vms"></a>Instrucciones de configuración de SAP Content Server para las instalaciones de SAP en máquinas virtuales de Azure
#### <a name="storage-configuration-for-content-server-in-azure"></a>Configuración de almacenamiento para el servidor de contenido de Azure
Si configura SAP Content Server para almacenar archivos en la base de datos de SAP MaxDB, todos los procedimientos recomendados de Azure Storage que se mencionan con relación a SAP MaxDB en este documento también son válidos en el caso de SAP Content Server. 

Si configura SAP Content Server para almacenar archivos en el sistema de archivos, se recomienda usar una unidad lógica exclusiva. El uso de espacios de almacenamiento de Windows también permite aumentar el tamaño del disco lógico y el rendimiento de IOPS, como se describe en el capítulo [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md). 

#### <a name="sap-content-server-location"></a>Ubicación de SAP Content Server
SAP Content Server se debe implementar en la misma región de Azure y de red virtual de Azure en la que se implementó el sistema SAP. Puede elegir si desea implementar los componentes de SAP Content Server en una máquina virtual de Azure dedicada o en la misma máquina virtual en la que se ejecuta el sistema SAP. 

![Máquina virtual de Azure dedicada para SAP Content Server](./media/dbms_maxdb_deployment_guide/800-azure-vm-sap-content-server.png)


#### <a name="sap-cache-server-location"></a>Ubicación del SAP Cache Server
El servidor SAP Cache Server es un componente adicional basado en servidores que proporciona acceso a documentos (en la memoria caché) en entornos locales. SAP Cache Server almacena en la caché los documentos de SAP Content Server. De este modo se optimiza el tráfico de red cuando los documentos se tienen que recuperar más de una vez desde distintas ubicaciones. Por lo general, SAP Cache Server debe encontrarse físicamente cerca del cliente que accede a este servidor. 

Dispone de dos opciones:

1. **El cliente es un sistema SAP de back-end** Si se configura un sistema SAP de back-end para acceder a SAP Content Server, dicho sistema SAP será un cliente. Puesto que el sistema SAP y SAP Content Server se implementan en la misma región de Azure, en el mismo centro de datos de Azure, ambos se encuentran físicamente próximos entre sí. Por lo tanto, no hace falta disponer de un SAP Content Server dedicado. Los clientes de interfaz de usuario de SAP (SAP GUI o un explorador web) acceden directamente al sistema SAP y este recupera los documentos desde SAP Content Server.
2. **El cliente es un explorador web local** SAP Content Server puede configurarse para acceder a él directamente mediante un explorador web. En este caso, un explorador web que se ejecute de forma local es un cliente de SAP Content Server. Se coloca un centro de datos local y el de Azure en diferentes ubicaciones físicas (lo ideal es que se encuentren próximos entre sí). Su centro de datos local se conecta a Azure a través del VPN de sitio a sitio o ExpressRoute de Azure. Aunque ambas opciones ofrecen conexión de red VPN segura a Azure, la conexión de red de sitio a sitio no ofrece un acuerdo de nivel de servicio de latencia y ancho de banda de red entre el centro de datos local y el de Azure. Para agilizar el acceso a los documentos, puede realizar una de las siguientes acciones:
   1. Instalar el servidor de caché de SAP de forma local, cerca del explorador web local (opción de ilustración a continuación)
   2. Configurar ExpressRoute de Azure, que ofrece una conexión de red dedicada de alta velocidad y baja latencia entre el centro de datos local y el centro de datos de Azure.

![Opción de instalar el servidor de caché de SAP de forma local](./media/dbms_maxdb_deployment_guide/900-sap-cache-server-on-premises.png)
<a name="642f746c-e4d4-489d-bf63-73e80177a0a8"></a>

#### <a name="backup--restore"></a>Copia de seguridad o restauración
Si configura SAP Content Server para almacenar archivos en la base de datos de SAP MaxDB, el procedimiento de copia de seguridad y restauración y las consideraciones sobre el rendimiento ya se describieron en los capítulos sobre SAP MaxDB de este documento. 

Si configura SAP Content Server para almacenar archivos en el sistema de archivos, una opción es ejecutar de forma manual la copia de seguridad o la restauración de toda la estructura de archivos donde se encuentran los documentos. Al igual que las copias de seguridad y las restauraciones de SAP MaxDB, se recomienda disponer de un volumen de disco dedicado para realizar copias de seguridad. 

#### <a name="other"></a>Otros
Otras configuraciones específicas de SAP Content Server son transparentes para máquinas virtuales de Azure y se describen en diversos documentos y notas de SAP:

* <https://service.sap.com/contentserver> 
* Nota de SAP [1619726]  
