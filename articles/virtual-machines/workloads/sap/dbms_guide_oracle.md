---
title: Implementación de DBMS de Azure Virtual Machines de Oracle para la carga de trabajo de SAP | Microsoft Docs
description: Implementación de DBMS de Azure Virtual Machines de Oracle para la carga de trabajo de SAP
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
ms.date: 12/14/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 15838e1e9acf328a0deaa981d1227c22c08dbbdf
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87832270"
---
# <a name="azure-virtual-machines-dbms-deployment-for-sap-workload"></a>Implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP

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


En este documento se describen las diferentes áreas que se deben tener en cuenta al implementar Oracle Database para la carga de trabajo de SAP en IaaS de Azure. Antes de leer este documento, le recomendamos leer [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md). También se recomienda leer otras guías que aparecen en la [documentación sobre la carga de trabajo de SAP en Azure](./get-started.md). 

Puede encontrar información sobre las versiones de Oracle y las versiones de SO correspondientes que se admiten para ejecutar SAP en Oracle en Azure en la nota de SAP [2039619].

Encontrará información general sobre cómo ejecutar SAP Business Suite en Oracle en [SAP en Oracle](https://www.sap.com/community/topic/oracle.html).
El software de Oracle está admitido por Oracle para su ejecución en Microsoft Azure. Para más información sobre la compatibilidad general con Azure y Windows Hyper-V, consulte el artículo con las [preguntas más frecuentes sobre Oracle y Microsoft Azure](https://www.oracle.com/technetwork/topics/cloud/faq-1963009.html). 

## <a name="sap-notes-relevant-for-oracle-sap-and-azure"></a>Notas de SAP pertinentes para Oracle, SAP y Azure 

Las siguientes notas de SAP están relacionadas con SAP en Azure.

| Número de nota | Título |
| --- | --- |
| [1928533] |SAP Applications on Azure: Supported Products and Azure VM types (Nota de SAP 1928533: Aplicaciones SAP en Azure: productos admitidos y tipos de máquina virtual de Azure) |
| [2015553] |SAP on Microsoft Azure: Support Prerequisites (Nota de soporte técnico 2015553 de SAP en Microsoft Azure: requisitos previos de soporte técnico) |
| [1999351] |Troubleshooting Enhanced Azure Monitoring for SAP (Solución de problemas de la supervisión mejorada de Azure para SAP) |
| [2178632] |Key Monitoring Metrics for SAP on Microsoft Azure (Métricas de supervisión clave para SAP en Microsoft Azure) |
| [2191498] |SAP on Linux with Azure: Enhanced Monitoring (SAP en Linux con Azure: supervisión mejorada) |
| [2039619] |SAP Applications on Microsoft Azure using the Oracle Database: Supported products and versions (Aplicaciones de SAP en Microsoft Azure con Base de datos de Oracle: versiones y productos compatibles) |
| [2243692] |Linux on Microsoft Azure (IaaS) VM: SAP license issues (Linux y máquinas virtuales de Microsoft Azure (IaaS): problemas de licencia de SAP) |
| [2069760] |Instalación y actualización de SAP en Oracle Linux 7.x |
| [1597355] |Recomendación de espacio de intercambio para Linux |
| [2171857] |Oracle Database 12c - compatibilidad con sistema de archivos en Linux |
| [1114181] |Oracle Database 11g - compatibilidad con sistema de archivos en Linux |

Las configuraciones exactas y la funcionalidad compatibles con Oracle y SAP en Azure se documentan en la nota de SAP [2039619](https://launchpad.support.sap.com/#/notes/2039619).

Windows y Oracle Linux son los únicos sistemas operativos compatibles con Oracle y SAP en Azure. Las populares distribuciones de Linux SLES y RHE no son compatibles con la implementación de los componentes de Oracle en Azure. Los componentes de Oracle incluyen el cliente de Oracle Database, que las aplicaciones de SAP usan para conectarse al DBMS de Oracle. 

Algunas excepciones, según la nota de SAP [2039619](https://launchpad.support.sap.com/#/notes/2039619), son los componentes de SAP que no usan el cliente de Oracle Database. Estos componentes de SAP son los servicios independientes de puesta en cola, servidor de mensajes, replicación de puesta en cola, webDispatcher y Gateway de SAP.  

Incluso si ejecuta el DBMS de Oracle y las instancias de aplicación de SAP en Oracle Linux, puede ejecutar los servicios centrales de SAP en SLES o RHEL y protegerlos con un clúster basado en Pacemaker. Pacemaker como plataforma de alta disponibilidad no es compatible con Oracle Linux.

## <a name="specifics-for-oracle-database-on-windows"></a>Detalles de Oracle Database en Windows

### <a name="oracle-configuration-guidelines-for-sap-installations-in-azure-vms-on-windows"></a>Directrices de configuración de Oracle para instalaciones de SAP en máquinas virtuales de Azure con Windows

De acuerdo con los manuales de instalación de SAP, los archivos relacionados con Oracle no deben instalarse ni ubicarse en el controlador del sistema para el disco del sistema operativo de una máquina virtual (unidad C:). Máquinas virtuales de distintos tamaños pueden admitir un número variable de discos conectados. Los tipos de máquinas virtuales más pequeños pueden admitir un menor número de discos conectados. 

Si tiene máquinas virtuales más pequeñas, se recomienda instalar o buscar Oracle home, satage, "saptrace", "saparch", "sapbackup", "sapcheck" o "sapreorg" en el disco del sistema operativo. Estas partes de los componentes del DBMS de Oracle no usan en gran medida la E/S y la capacidad de proceso. Esto significa que el disco del sistema operativo puede ocuparse de los requisitos de E/S. El tamaño predeterminado del disco del sistema operativo es de 127 GB. 

Si no hay espacio libre disponible suficiente, se puede [cambiar el tamaño](../../windows/expand-os-disk.md) del disco a 2048 GB. Los archivos de registro de puesta al día y Oracle Database tienen que almacenarse en discos de datos independientes. Hay una excepción con el espacio de tablas temporales de Oracle. Los archivos temporales se pueden crear en la unidad D:/ (unidad no persistente). La unidad D:\ no persistente también ofrece un mejor rendimiento y latencia de E/S (a excepción de las máquinas virtuales de la serie A). 

Para determinar la cantidad de espacio correcta para los archivos temporales, puede revisar el tamaño de los archivos temporales de los sistemas existentes.

### <a name="storage-configuration"></a>Configuración de almacenamiento
Solo se admiten versiones de Oracle de solo una versión que usen discos con formato NTFS. Todos los archivos de la base de datos se deben almacenar en el sistema de archivos NTFS en discos VHD o Managed Disks (recomendado). Estos discos se montan en la máquina virtual de Azure y se basan en [Azure Page BLOB Storage](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs) o en [Azure Managed Disks](../../windows/managed-disks-overview.md). 

Se recomienda [Azure Managed Disks](../../windows/managed-disks-overview.md). También se recomienda encarecidamente usar [discos SSD Premium](../../windows/disks-types.md) para las implementaciones de Oracle Database.

Las unidades de red o los recursos compartidos remotos, como los servicios de archivos de Azure, no son compatibles con los archivos de Oracle Database. Para más información, consulte:

- [Introducing Microsoft Azure File Service (Introducción al servicio de archivos de Microsoft Azure)](/archive/blogs/windowsazurestorage/introducing-microsoft-azure-file-service)

- [Persisting connections to Microsoft Azure Files (Persistencia de conexiones en archivos de Microsoft Azure)](/archive/blogs/windowsazurestorage/persisting-connections-to-microsoft-azure-files)


Si se usan discos basados en Azure Page BLOB Storage o Managed Disks, las declaraciones realizadas en [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md) también se aplican a las implementaciones con Oracle Database.

Existen cuotas en el rendimiento de IOPS para los discos de Azure. Este concepto se explica en [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md). Las cuotas exactas varían según el tipo de máquina virtual que se usa. Encontrará una lista de los tipos de máquina virtual con sus cuotas en [Tamaños de las máquinas virtuales Windows en Azure][virtual-machines-sizes-windows].

Para identificar los tipos de máquina virtual de Azure compatibles, consulte la nota de SAP [1928533].

La configuración mínima es la siguiente: 

| Componente | Disco | Almacenamiento en memoria caché | Bloque de almacenamiento |
| --- | ---| --- | --- |
| \oracle\<SID>\origlogaA & mirrlogB | Premium | None | No es necesario |
| \oracle\<SID>\origlogaB & mirrlogA | Premium | None | No es necesario |
| \oracle\<SID>\sapdata1...n | Premium | Solo lectura | Se puede usar |
| \oracle\<SID>\oraarch | Estándar | None | No es necesario |
| Oracle Home, saptrace, ... | Disco del sistema operativo | | No es necesario |


La selección de discos para hospedar los registros de la fase de puesta al día en línea debe basarse en los requisitos de IOPS. Es posible almacenar todos los sapdata1…n (espacios de tabla) en un solo disco montado, siempre que el tamaño, IOPS y capacidad de proceso cumplan los requisitos. 

La configuración de rendimiento es la siguiente:

| Componente | Disco | Almacenamiento en memoria caché | Bloque de almacenamiento |
| --- | ---| --- | --- |
| \oracle\<SID>\origlogaA | Premium | None | Se puede usar  |
| \oracle\<SID>\origlogaB | Premium | None | Se puede usar |
| \oracle\<SID>\mirrlogAB | Premium | None | Se puede usar |
| \oracle\<SID>\mirrlogBA | Premium | None | Se puede usar |
| \oracle\<SID>\sapdata1...n | Premium | Solo lectura | Recomendado  |
| \oracle\SID\sapdata(n+1)* | Premium | None | Se puede usar |
| \oracle\<SID>\oraarch* | Premium | None | No es necesario |
| Oracle Home, saptrace, ... | Disco del sistema operativo | No es necesario |

*(n+1): hospedar espacios de tablas SYSTEM, TEMP y UNDO. El patrón de E/S de los espacios de tablas System y Undo son diferentes de otros espacios de tabla que hospedan los datos de aplicación. No almacenar en caché es la mejor opción para el rendimiento de los espacios de tablas System y Undo.

*oraarch: no es necesario el bloque de almacenamiento desde un punto de vista del rendimiento. Se puede usar para obtener más espacio.

Si se necesita más IOPS, se recomienda usar bloques de almacenamiento de Windows (solo disponibles en Windows Server 2012 y versiones posteriores) para crear un dispositivo lógico de gran tamaño en varios discos montados. Este enfoque simplifica la sobrecarga de administración para administrar el espacio en disco y ayuda a evitar el esfuerzo de distribuir archivos manualmente en varios discos montados.


#### <a name="write-accelerator"></a>Acelerador de escritura
En las máquinas virtuales de Azure de la serie M, se puede reducir la latencia de escritura en los registros de fase de puesta al día en línea, en comparación con el rendimiento de Azure Premium Storage. Habilite el Acelerador de escritura de Azure para los discos (discos duros virtuales) en función del Azure Premium Storage que se usa para los archivos de registro de puesta al día en línea. Para más información, consulte [Acelerador de escritura](../../linux/how-to-enable-write-accelerator.md).


### <a name="backuprestore"></a>Copia de seguridad y restauración
Para la funcionalidad de copia de seguridad y restauración, las SAP BR*Tools para Oracle se admiten del mismo modo que en los sistemas operativos Windows Server estándar. También se admite Oracle Recovery Manager (RMAN) para las copias de seguridad en disco y las restauraciones desde disco.

También se puede usar el servicio Azure Backup para ejecutar una copia de seguridad de máquina virtual coherente con la aplicación. En el artículo [Planeación de la infraestructura de copia de seguridad de máquinas virtuales en Azure](../../../backup/backup-azure-vms-introduction.md) se explica cómo Azure Backup usa la funcionalidad de Windows VSS para ejecutar copias de seguridad coherentes con la aplicación. Las versiones de Oracle Database que se admiten en Azure y SAP pueden aprovechar la funcionalidad VSS para las copias de seguridad. Para más información, consulte [Basic concepts of database backup and recovery with VSS](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/ntqrf/basic-concepts-of-database-backup-and-recovery-with-vss.html#GUID-C085101B-237F-4773-A2BF-1C8FD040C701) (Conceptos básicos de copia de seguridad y recuperación de bases de datos con VSS) en la documentación de Oracle.


### <a name="high-availability"></a>Alta disponibilidad
Oracle Data Guard se admite con fines de alta disponibilidad y recuperación ante desastres. Para lograr una conmutación por error automática en Data Guard, debe usar Fast-Start Failover (FSFA). La funcionalidad Observador (FSFA) desencadena la conmutación por error. Si no usa FSFA, solo puede usar una configuración de conmutación por error manual.

Para más información sobre la recuperación ante desastres de bases de datos de Oracle en Azure, consulte [Recuperación ante desastres para Oracle Database 12c en el entorno de Azure](../oracle/oracle-disaster-recovery.md).

### <a name="accelerated-networking"></a>Redes aceleradas
En el caso de las implementaciones de Oracle en Windows, se recomienda encarecidamente las redes aceleradas tal como se describe en [Redes aceleradas de Azure](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/). Tenga en cuenta también las recomendaciones realizadas en [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md). 
### <a name="other"></a>Otros
En [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md), se describen otros conceptos importantes relacionados con las implementaciones de máquinas virtuales con Oracle Database, incluidos conjuntos de disponibilidad de Azure y la supervisión de SAP.

## <a name="specifics-for-oracle-database-on-oracle-linux"></a>Detalles de Oracle Database en Oracle Linux
El software de Oracle está admitido por Oracle para su ejecución en Microsoft Azure con Oracle Linux como el sistema operativo invitado. Para más información sobre la compatibilidad general con Windows Hyper-V y Azure, consulte el artículo con las [preguntas más frecuentes sobre Azure y Oracle](https://www.oracle.com/technetwork/topics/cloud/faq-1963009.html). 

También se admite el escenario concreto en el que las aplicaciones de SAP aprovechen las instancias de Oracle Database. Los detalles se mencionan en la siguiente parte del documento.

### <a name="oracle-version-support"></a>Compatibilidad de versiones de Oracle
Para información sobre las versiones de Oracle y las versiones de SO correspondientes compatibles con la ejecución de SAP en Oracle en Azure Virtual Machines, consulte la nota de SAP [2039619].

Encontrará información general sobre cómo ejecutar SAP Business Suite en Oracle en la [página de la comunidad de SAP en Oracle](https://www.sap.com/community/topic/oracle.html).

### <a name="oracle-configuration-guidelines-for-sap-installations-in-azure-vms-on-linux"></a>Directrices de configuración de Oracle para instalaciones de SAP en máquinas virtuales de Azure con Linux

De acuerdo con los manuales de instalación de SAP, los archivos relacionados con Oracle no deben instalarse ni ubicarse en el controlador del sistema del disco de arranque de la máquina virtual. Máquinas virtuales de distintos tamaños pueden admitir un número variable de discos conectados. Los tipos de máquinas virtuales más pequeños pueden admitir un menor número de discos conectados. 

En este caso, se recomienda instalar o buscar Oracle home, stage, saptrace, saparch, sapbackup, sapcheck, sapreorg para el disco de arranque. Estas partes de los componentes del DBMS de Oracle no usan en gran medida la E/S y la capacidad de proceso. Esto significa que el disco del sistema operativo puede ocuparse de los requisitos de E/S. El tamaño predeterminado del disco del sistema operativo es de 30 GB. Puede expandir el disco de arranque con Azure Portal, PowerShell o la CLI de Azure. Una vez que se expande el disco de arranque, puede agregar una partición adicional para los binarios de Oracle.


### <a name="storage-configuration"></a>Configuración de almacenamiento

Los sistemas de archivos ext4, xfs u Oracle ASM se admiten para los archivos de Oracle Database en Azure. Todos los archivos de la base de datos se deben almacenar en estos sistemas de archivos basados en discos VHD o Managed Disks. Estos discos se montan en la máquina virtual de Azure y se basan en [Azure Page BLOB Storage](<https://docs.microsoft.com/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs>) o en [Azure Managed Disks](../../windows/managed-disks-overview.md).

Para los kernels de Oracle Linux UEK, se requiere como mínimo la versión 4 de UEK para que sea compatible con los [discos SSD Azure Premium](../../windows/premium-storage-performance.md#disk-caching).

Se recomienda encarecidamente el uso de [Azure Managed Disks](../../windows/managed-disks-overview.md). También se recomienda encarecidamente usar [discos SSD Azure Premium](../../windows/disks-types.md) para las implementaciones de Oracle Database.

Las unidades de red o los recursos compartidos remotos, como los servicios de archivos de Azure, no son compatibles con los archivos de Oracle Database. Para obtener más información, vea lo siguiente: 

- [Introducing Microsoft Azure File Service (Introducción al servicio de archivos de Microsoft Azure)](/archive/blogs/windowsazurestorage/introducing-microsoft-azure-file-service)

- [Persisting connections to Microsoft Azure Files (Persistencia de conexiones en archivos de Microsoft Azure)](/archive/blogs/windowsazurestorage/persisting-connections-to-microsoft-azure-files)

Si se usan discos basados en Azure Page BLOB Storage o Managed Disks, las declaraciones realizadas en [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md) también se aplican a las implementaciones con Oracle Database.

 Existen cuotas en el rendimiento de IOPS para los discos de Azure. Este concepto se explica en [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md). Las cuotas exactas depende del tipo de máquina virtual que se usa. Para ver una lista de los tipos de máquinas virtuales con sus cuotas, consulte [Tamaños de las máquinas virtuales Linux en Azure][virtual-machines-sizes-linux].

Para identificar los tipos de máquina virtual de Azure compatibles, consulte la nota de SAP [1928533].

Configuración mínima:

| Componente | Disco | Almacenamiento en memoria caché | Recorte* |
| --- | ---| --- | --- |
| /oracle/\<SID>/origlogaA & mirrlogB | Premium | None | No es necesario |
| /oracle/\<SID>/origlogaB & mirrlogA | Premium | None | No es necesario |
| /oracle/\<SID>/sapdata1...n | Premium | Solo lectura | Se puede usar |
| /oracle/\<SID>/oraarch | Estándar | None | No es necesario |
| Oracle Home, saptrace, ... | Disco del sistema operativo | | No es necesario |

*Eliminación: franja LVM o MDADM mediante RAID0

La selección de discos para hospedar los registros de la fase de puesta al día en línea de Oracle debe basarse en los requisitos de IOPS. Es posible almacenar todos los sapdata1…n (espacios de tabla) en un solo disco montado, siempre que el volumen, IOPS y capacidad de proceso cumplan los requisitos. 

Configuración del rendimiento:

| Componente | Disco | Almacenamiento en memoria caché | Recorte* |
| --- | ---| --- | --- |
| /oracle/\<SID>/origlogaA | Premium | None | Se puede usar  |
| /oracle/\<SID>/origlogaB | Premium | None | Se puede usar |
| /oracle/\<SID>/mirrlogAB | Premium | None | Se puede usar |
| /oracle/\<SID>/mirrlogBA | Premium | None | Se puede usar |
| /oracle/\<SID>/sapdata1...n | Premium | Solo lectura | Recomendado  |
| /oracle/\<SID>/sapdata(n+1)* | Premium | None | Se puede usar |
| /oracle/\<SID>/oraarch* | Premium | None | No es necesario |
| Oracle Home, saptrace, ... | Disco del sistema operativo | No es necesario |

*Eliminación: franja LVM o MDADM mediante RAID0

*(n+1): hospedar espacios de tablas SYSTEM, TEMP y UNDO: El patrón de E/S de los espacios de tablas System y Undo son diferentes de otros espacios de tabla que hospedan los datos de aplicación. No almacenar en caché es la mejor opción para el rendimiento de los espacios de tablas System y Undo.

*oraarch: no es necesario el bloque de almacenamiento desde un punto de vista del rendimiento.


Si se requieren más IOPS, se recomienda usar LVM (administrador de volúmenes lógicos) o MDADM para crear un volumen lógico de gran tamaño en varios discos montados. Para más información, consulte el documento [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md) en relación con las directrices e indicadores sobre cómo aprovechar LVM o MDADM. Este enfoque simplifica la sobrecarga de administración para administrar el espacio en disco y ayuda a evitar el esfuerzo de distribuir archivos manualmente en varios discos montados.


#### <a name="write-accelerator"></a>Acelerador de escritura
En las máquinas virtuales de Azure de la serie M, cuando se usa el Acelerador de escritura de Azure se puede reducir la latencia de escritura en los registros de fase de puesta al día en línea, en comparación con el rendimiento de Azure Premium Storage. Habilite el Acelerador de escritura de Azure para los discos (discos duros virtuales) en función del Azure Premium Storage que se usa para los archivos de registro de puesta al día en línea. Para más información, consulte [Acelerador de escritura](../../linux/how-to-enable-write-accelerator.md).


### <a name="backuprestore"></a>Copia de seguridad y restauración
Para utilizar la funcionalidad de copia de seguridad y restauración, se admite SAP BR*Tools for Oracle del mismo modo que en un servidor físico e Hyper-V. También se admite Oracle Recovery Manager (RMAN) para las copias de seguridad en disco y las restauraciones desde disco.

Para más información sobre cómo puede usar los servicios Azure Backup y Recovery para crear una copia de seguridad de las bases de datos de Oracle y recuperarlas, consulte el artículo [Copia de seguridad y recuperación de una base de datos de Oracle Database 12c en una máquina virtual Linux de Azure](../oracle/oracle-backup-recovery.md).

### <a name="high-availability"></a>Alta disponibilidad
Oracle Data Guard se admite con fines de alta disponibilidad y recuperación ante desastres. Para lograr una conmutación por error automática en Data Guard, debe usar Fast-Start Failover (FSFA). La funcionalidad Observador (FSFA) desencadena la conmutación por error. Si no usa FSFA, solo puede usar una configuración de conmutación por error manual. Para más información, consulte [Implementación de Oracle Data Guard en una máquina virtual Linux en Azure](../oracle/configure-oracle-dataguard.md).


Los aspectos de la recuperación ante desastres para bases de datos de Oracle en Azure se presentan en el artículo [Recuperación ante desastres para Oracle Database 12c en el entorno de Azure](../oracle/oracle-disaster-recovery.md).

### <a name="accelerated-networking"></a>Redes aceleradas
La compatibilidad con Azure Accelerated Networking en Oracle Linux se ofrece con Oracle Linux 7 Update 5 (Oracle Linux 7.5). Si no puede actualizar a la versión más reciente de Oracle Linux 7.5, es posible que haya una solución alternativa si se usa RedHat Compatible Kernel (RHCK) en lugar del kernel UEK de Oracle. 

El uso del kernel RHEL en Oracle Linux se admite según la nota de SAP [1565179](https://launchpad.support.sap.com/#/notes/1565179). En el caso de las redes aceleradas de Azure, la versión mínima del kernel RHCKL debe ser 3.10.0-862.13.1.el7. Si usa el kernel de UEK en Oracle Linux junto con las [redes aceleradas de Azure](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/), necesita usar el kernel de Oracle UEK versión 5.

Si va a implementar máquinas virtuales desde una imagen que no está basada en Azure Marketplace, necesitará copiar archivos de configuración adicionales en la máquina virtual mediante la ejecución del código siguiente: 
<pre><code># Copy settings from GitHub to the correct place in the VM
sudo curl -so /etc/udev/rules.d/68-azure-sriov-nm-unmanaged.rules https://raw.githubusercontent.com/LIS/lis-next/master/hv-rhel7.x/hv/tools/68-azure-sriov-nm-unmanaged.rules 
</code></pre>


### <a name="other"></a>Otros
En [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md), se describen otros conceptos importantes relacionados con las implementaciones de máquinas virtuales con Oracle Database, incluidos conjuntos de disponibilidad de Azure y la supervisión de SAP.