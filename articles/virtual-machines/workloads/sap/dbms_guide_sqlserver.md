---
title: Implementación de DBMS de Azure Virtual Machines de SQL Server para la carga de trabajo de SAP | Microsoft Docs
description: Implementación de DBMS de Azure Virtual Machines de SQL Server para la carga de trabajo de SAP
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
ms.date: 09/26/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 6e217540b1dd3744da855c71e0add289dd1c9e18
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87831063"
---
# <a name="sql-server-azure-virtual-machines-dbms-deployment-for-sap-netweaver"></a>Implementación de DBMS de Azure Virtual Machines de SQL Server para la carga de trabajo de SAP NetWeaver

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

[dbms-guide]:dbms-guide_general.md 
[dbms-guide-2.1]:dbms-guide.md#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f 
[dbms-guide-2.2]:dbms-guide.md#c8e566f9-21b7-4457-9f7f-126036971a91 
[dbms-guide-2.3]:dbms-guide.md#10b041ef-c177-498a-93ed-44b3441ab152 
[dbms-guide-2]:dbms-guide.md#65fa79d6-a85f-47ee-890b-22e794f51a64 
[dbms-guide-3]:dbms-guide.md#871dfc27-e509-4222-9370-ab1de77021c3 
[dbms-guide-5.5.1]:dbms-guide.md#0fef0e79-d3fe-4ae2-85af-73666a6f7268 
[dbms-guide-5.5.2]:dbms-guide.md#f9071eff-9d72-4f47-9da4-1852d782087b 
[dbms-guide-5.6]:dbms_guide_sqlserver.md#1b353e38-21b3-4310-aeb6-a77e7c8e81c8 
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
[virtual-machines-workload-template-sql-alwayson]:https://azure.microsoft.com/eresources/templates/sql-server-2014-alwayson-existing-vnet-and-ad/
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




En este documento se describen las diferentes áreas que se deben tener en cuenta al implementar SQL Server para la carga de trabajo de SAP en IaaS de Azure. Como condición previa a este documento, debe haber leído el documento [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md), así como otras guías de la [documentación de carga de trabajo de SAP en Azure](./get-started.md). 



> [!IMPORTANT]
> El ámbito de este documento es la versión de Windows en SQL Server. SAP no es compatible con la versión de Linux de SQL Server con ningún software de SAP. En el documento no se habla de Microsoft Azure SQL Database, que es la oferta de plataforma como servicio de Microsoft Azure. En este documento se aborda la ejecución del producto SQL Server tal cual para implementaciones locales en Azure Virtual Machines, con lo que se aprovecha la funcionalidad de infraestructura como servicio. Las funcionalidades de bases de datos de estas dos ofertas son diferentes y no deben combinarse. Consulte también: <https://azure.microsoft.com/services/sql-database/>
> 
>

En general, debería considerar usar las versiones más recientes de SQL Server para ejecutar la carga de trabajo de SAP en IaaS de Azure. Las versiones más recientes de SQL Server ofrecen una mejor integración en algunos de los servicios y funcionalidades de Azure o disponer de cambios que optimicen las operaciones en una infraestructura de IaaS de Azure.

Se recomienda revisar [esta][virtual-machines-sql-server-infrastructure-services] documentación antes de continuar.

En las secciones siguientes se agregan y mencionan fragmentos de partes de la documentación del vínculo anterior. Se describen de forma más detallada algunos conceptos y se mencionan consideraciones sobre SAP. Sin embargo, se recomienda encarecidamente consultar la documentación anterior primero antes de leer la documentación específica de SQL Server.

Debe conocer información específica sobre SQL Server en IaaS antes de continuar:

* **Soporte para versiones de SQL**: para los clientes de SAP se presta soporte para SQL Server 2008 R2 y posteriores en las máquinas virtuales de Microsoft Azure. No se prestará soporte para versiones anteriores. Revise esta [declaración de soporte](https://support.microsoft.com/kb/956893) general para más información. En general, Microsoft también presta soporte para SQL Server 2008. Sin embargo, debido a la importante funcionalidad de SAP que se introdujo con SQL Server 2008 R2, SQL Server 2008 R2 es la versión mínima que se requiere para SAP. En general, debería considerar usar las versiones más recientes de SQL Server para ejecutar la carga de trabajo de SAP en IaaS de Azure. Las versiones más recientes de SQL Server ofrecen una mejor integración en algunos de los servicios y funcionalidades de Azure o disponer de cambios que optimicen las operaciones en una infraestructura de IaaS de Azure. Por lo tanto, el documento se limita a SQL Server 2016 y SQL Server 2017.
* **Rendimiento de SQL**: las máquinas virtuales hospedadas en Microsoft Azure funcionan bien en comparación con otras ofertas de virtualización de nube pública, aunque los resultados individuales pueden variar. Consulte el artículo [Procedimientos recomendados para SQL Server en Azure Virtual Machines](../../../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md).
* **Uso de imágenes de Azure Marketplace**: la forma más rápida de implementar una nueva máquina virtual de Microsoft Azure es utilizando una imagen de Azure Marketplace. Hay imágenes en Azure Marketplace que contienen las versiones más recientes de SQL Server. Las imágenes donde ya se ha instalado SQL Server no se pueden utilizar inmediatamente para aplicaciones de SAP NetWeaver. El motivo es que la intercalación de SQL Server predeterminada está instalada en esas imágenes y no la que requieren los sistemas SAP NetWeaver. Para usar estas imágenes, consulte los pasos que se explican en el capítulo [Uso de imágenes de SQL Server desde Microsoft Azure Marketplace][dbms-guide-5.6]. 


## <a name="recommendations-on-vmvhd-structure-for-sap-related-sql-server-deployments"></a>Recomendaciones sobre la estructura de máquina virtual y VHD para SAP relacionadas con las implementaciones de SQL Server
Según la descripción general, los archivos ejecutables de SQL Server se deben ubicar o instalar en la unidad del sistema del disco de sistema operativo de la máquina virtual (unidad C:\).  Normalmente, la carga de trabajo de SAP NetWeaver no utiliza en gran medida la mayoría de las bases de datos de sistema de SQL Server. Como resultado, las bases de datos de sistema de SQL Server (master, msdb y model) pueden permanecer también en la unidad C:\. Una excepción podría ser tempdb, que, en el caso de las cargas de trabajo de SAP, podría requerir un mayor volumen de datos o un mayor volumen de operaciones de E/S. La carga de trabajo de E/S no se debe aplicar al disco duro virtual de sistema operativo. Para estos sistemas, se deben realizar los pasos siguientes:


* Con todos los tipos de máquina virtual certificados de SAP (consulte la nota de SAP [1928533]), salvo las máquinas virtuales de la serie A, los datos de tempdb y los archivos de registro se pueden colocar en la unidad D:\ no persistente. 
* No obstante, se recomienda usar varios archivos de datos de tempdb. Tenga en cuenta que los volúmenes de la unidad D:\ serán diferentes en función del tipo de máquina virtual. Para saber cuáles son los tamaños exactos de la unidad D:\ de las distintas máquinas virtuales, consulte el artículo [Tamaños de las máquinas virtuales Windows en Azure](../../sizes.md).

Estas configuraciones permiten que tempdb consuma más espacio que el que la unidad del sistema puede proporcionar. La unidad D:\ no persistente también ofrece un mejor rendimiento y latencia de E/S (a excepción de las máquinas virtuales de la serie A). Para determinar el tamaño correcto de tempdb, puede comprobar los tamaños de tempdb en los sistemas existentes. 

>[!NOTE]
> En el caso de que coloque los archivos de datos de tempdb y el archivo de registro en una carpeta de la unidad D:\ que haya creado, deberá asegurarse de que la carpeta existe después de reiniciar la máquina virtual. Dado que la unidad D:\ se acaba de inicializar después de reiniciar la máquina virtual, se borrarán todas las estructuras de archivos y directorios. En [este artículo](https://cloudblogs.microsoft.com/sqlserver/2014/09/25/using-ssds-in-azure-vms-to-store-sql-server-tempdb-and-buffer-pool-extensions/) se documenta una posibilidad de volver a crear estructuras de directorios finales en la unidad D:\ antes de iniciar el servicio SQL Server.

Una configuración de máquina virtual que ejecuta SQL Server con una base de datos de SAP donde el archivo de registro de tempdb y el de datos se colocan en la unidad D:\ tendría el siguiente aspecto:

![Diagrama de una configuración simple de disco de máquina virtual para SQL Server](./media/dbms_sqlserver_deployment_guide/Simple_disk_structure.PNG)

En el diagrama anterior se muestra un caso sencillo. Como se menciona en el artículo [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md), el número y el tamaño de los discos de Premium Storage dependen de varios factores, pero en general se recomienda lo siguiente:

- Usar espacios de almacenamiento para formar uno o una cifra pequeña de volúmenes, que contienen los archivos de datos de SQL Server. El motivo de esta configuración es que en la vida real hay muchas bases de datos de SAP con archivos de base de datos de distintos tamaños y con diferentes cargas de trabajo de E/S.
- Usar espacios de almacenamiento para proporcionar IOPS suficientes y para el archivo de registro de transacciones de SQL Server. La carga de trabajo IOPS potencial es a menudo el hilo conductor para definir el tamaño del volumen de registro de transacciones, y no del volumen potencial del volumen de transacciones de SQL Server.
- Usar la unidad D:\ para tempdb mientras el rendimiento sea lo suficientemente bueno. Si la carga de trabajo global está limitada en rendimiento por el hecho de que tempdb se encuentra en la unidad D:\, puede que deba tener en cuenta trasladar tempdb a discos de Premium Storage independientes, como se recomienda en [este artículo](../../../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md).


### <a name="special-for-m-series-vms"></a>Aspectos especiales para las máquinas virtuales de la serie M
Para la máquina virtual de Azure de la serie M, se puede reducir la latencia de escritura en el registro de transacciones mediante factores, en comparación con el rendimiento de Azure Premium Storage, cuando se usa el Acelerador de escritura de Azure. Por tanto, se debe implementar el Acelerador de escritura de Azure para los discos duros virtuales que forman el volumen para el registro de transacciones de SQL Server. En el documento [Acelerador de escritura](../../windows/how-to-enable-write-accelerator.md) se pueden leer los detalles.
  

### <a name="formatting-the-disks"></a>Aplicar formato a los discos
Para SQL Server, el tamaño de bloque NTFS de los discos que contienen los archivos de registro y de datos de SQL Server deben tener un tamaño de 64 KB. No hay que formatear la unidad D:\, ya que este proceso se ha realizado previamente.

Para asegurarse de que la restauración o la creación de bases de datos no inicialice los archivos de datos llenando con ceros el contenido de los archivos, debe asegurarse de que el servicio de SQL Server que se ejecuta en el contexto de usuario tiene un permiso determinado. Normalmente, los usuarios del grupo de administrador de Windows tienen estos permisos. Si el servicio SQL Server se ejecuta en el contexto de usuario del usuario no administrador de Windows, debe asignar a ese usuario el derecho de usuario **Realizar tareas de mantenimiento del volumen**.  Consulte los detalles en este artículo de Microsoft Knowledge Base: <https://support.microsoft.com/kb/2574695>

### <a name="impact-of-database-compression"></a>Impacto de la compresión de las bases de datos
En configuraciones donde el ancho de banda de E/S puede convertirse en un factor de limitación, todas las medidas que reduzcan el número de IOPS podrían ayudar a ajustar la carga de trabajo que se puede ejecutar en un escenario de IaaS como Azure. Por lo tanto, si aún no lo ha hecho, Microsoft y SAP recomiendan que aplique la compresión de página de SQL Server antes de cargar bases de datos de SAP existentes en Azure.

La recomendación para realizar la compresión de la base de datos antes de cargar en Azure viene motivada por dos razones:

* La cantidad de datos que se carga es menor.
* La duración de la ejecución de la compresión es más breve dándose por supuesto que se puede utilizar hardware más eficaz con más CPU, mayor ancho de banda de E/S o menos latencia de E/S en un entorno local.
* Si se utilizan tamaños de base de datos más pequeños, se incurriría en menos costes de asignación de disco.

La compresión de las bases de datos funciona también correctamente en Azure Virtual Machines de la misma forma que en local. Para obtener más información sobre cómo comprimir bases de datos de SQL Server de SAP NetWeaver, consulte el artículo [Improved SAP compression tool MSSCOMPRESS](/archive/blogs/saponsqlserver/improved-sap-compression-tool-msscompress) (Herramienta de compresión de SAP MSSCOMPRESS mejorada). 

## <a name="sql-server-2014-and-more-recent---storing-database-files-directly-on-azure-blob-storage"></a>SQL Server 2014 y versiones más recientes: almacenamiento de archivos de base de datos directamente en Azure Blob Storage
SQL Server 2014 y versiones posteriores ofrecen la posibilidad de almacenar archivos de base de datos directamente en el almacén de blobs de Azure sin necesidad de usar el contenedor de un disco duro virtual en ellos. Especialmente al usar Standard Azure Storage o tipos de máquina virtual con tamaños más pequeños, este tipo de implementación da lugar a escenarios donde pueden superarse los límites de IOPS impuestos por un número limitado de discos que pueden montarse en tipos de máquinas virtuales de menor tamaño. Aunque este método de implementación funciona con las bases de datos de usuario, no funciona con las de sistema de SQL Server. También funciona con los archivos de registro y de datos de SQL Server. Si quiere implementar una base de datos SQL Server de SAP de este modo, en lugar de contenerla en discos, debe tener en cuenta lo siguiente:

* La cuenta de almacenamiento utilizada debe estar en la misma región de Azure que la que se empleó para implementar la máquina virtual que se está ejecutando en SQL Server.
* Para este método de implementaciones también se aplican las consideraciones que se indicaron anteriormente relacionadas con la distribución de los discos virtuales en diferentes cuentas de Azure Storage. Significa que el número de operaciones de E/S cuenta para los límites de la cuenta de Azure Storage.
* En lugar de contabilizarse en la cuota de E/S de almacenamiento de la máquina virtual, el tráfico de los blobs de almacenamiento que representan los datos y los archivos de registro de SQL Server se contabilizará en el ancho de banda de red de la máquina virtual del tipo de máquina virtual concreto. Para saber cuál es la red y el ancho de banda de red de un determinado tipo de máquina virtual, consulte el artículo [Tamaños de las máquinas virtuales Windows en Azure](../../sizes.md).
* Como resultado de la inserción de E/S de archivos mediante la cuota de red, se eliminará mayormente la cuota de almacenamiento y, con ese uso, solo parcialmente el ancho de banda de red general de la máquina virtual.
* Ya no se aplican los objetivos de rendimiento de E/S y de IOPS que tiene Azure Premium Storage para los distintos tamaños de disco. Incluso si los blobs que ha creado están en Azure Premium Storage. Los objetivos están documentados en el artículo [Discos administrados y Premium Storage de alto rendimiento para VM](../../windows/disks-types.md#premium-ssd). Como resultado de colocar los archivos de datos y los archivos de registro de SQL Server directamente en blobs que se almacenan en Azure Premium Storage, las características de rendimiento pueden variar respecto a los discos duros virtuales de Azure Premium Storage.
* El almacenamiento en caché basado en host que está disponible para los discos de Azure Premium Storage no está disponible al colocar los archivos de datos de SQL Server directamente en los blobs de Azure.
* En las máquinas virtuales de la serie M no se puede usar el Acelerador de escritura de Azure para admitir escrituras inferiores a milisegundos en el archivo de registro de transacciones de SQL Server. 

En el artículo [Archivos de datos de SQL Server en Microsoft Azure](/sql/relational-databases/databases/sql-server-data-files-in-microsoft-azure?view=sql-server-2017) encontrará información detallada sobre esta funcionalidad.

La recomendación para los sistemas de producción consiste en evitar esta configuración y elegir las ubicaciones de los datos y los archivos de registro de SQL Server en discos duros virtuales de Azure Premium Storage en lugar de colocarlos directamente en blobs de Azure.


## <a name="sql-server-2014-buffer-pool-extension"></a>Extensión de grupo de búferes de SQL Server 2014
SQL Server 2014 incorporó una nueva característica denominada [Extensión del grupo de búferes](/sql/database-engine/configure-windows/buffer-pool-extension?view=sql-server-2017). Esta funcionalidad amplía el grupo de búferes de SQL Server que se mantiene en memoria con una memoria caché de segundo nivel respaldada por discos SSD locales de un servidor o una máquina virtual. La extensión del grupo de búferes permite tener "en memoria" un espacio de trabajo de datos mayor. En comparación con el acceso a Azure Standard Storage, el acceso a la extensión del grupo de búferes que se almacena en discos SSD locales de una máquina virtual de Azure es mucho más rápido. Si se compara la Extensión del grupo de búferes con la Caché de lectura de Azure Premium Storage, como se recomienda para los archivos de datos de SQL Server, no se esperan ventajas significativas para las extensiones del grupo de búferes. El motivo es que las memorias caché (extensión de grupo de búferes de SQL Server y memoria caché de lectura de Premium Storage) usan los discos locales del nodo de proceso de Azure.

La experiencia adquirida mientras tanto con la Extensión del grupo de búferes de SQL Server con la carga de trabajo de SAP es variada y aún no ofrece unas recomendaciones claras sobre si se debe usar en todos los casos. Lo ideal es que el espacio de trabajo que requiere la aplicación de SAP se adapte a la memoria principal. Con la oferta de Azure de máquinas virtuales de hasta 4 TB de memoria, debería ser factible mantener el espacio de trabajo en memoria. Por lo tanto, el uso de la Extensión del grupo de búferes se limita a algunos casos excepcionales y no debería ser un caso convencional.  

## <a name="backuprecovery-considerations-for-sql-server"></a>Consideraciones de copia de seguridad y recuperación en SQL Server
Al implementar SQL Server en Azure, se debe revisar la metodología de copia de seguridad. Aunque no se trate de un sistema de producción, debe llevarse a cabo de manera periódica una copia de seguridad de la base de datos de SAP hospedada por SQL Server. Como Azure Storage mantiene tres imágenes, una copia de seguridad es menos importante para compensar un bloqueo de almacenamiento. El principal motivo para mantener un plan de copia de seguridad y recuperación adecuado es más que poder compensar los errores lógicos o manuales proporcionando funcionalidades de recuperación a un momento dado. El objetivo es usar las copias de seguridad para restaurar la base de datos a un momento dado o utilizar las copias de seguridad de Azure para propagar otro sistema mediante la copia de la base de datos existente. 

Para buscar otras posibilidades de copia de seguridad de SQL Server en Azure, lea el artículo [Copias de seguridad y restauración para SQL Server en Azure Virtual Machines](../../../azure-sql/virtual-machines/windows/backup-restore.md). En este artículo se describen varias posibilidades.

### <a name="manual-backups"></a>Copias de seguridad manuales
Tiene varias posibilidades para hacer copias de seguridad "manuales":

1. Realizar copias de seguridad convencionales de SQL Server en discos de Azure conectados directos. Este método tiene la ventaja de que tiene a su disposición las copias de seguridad con rapidez para las actualizaciones del sistema y la creación de nuevos sistemas como copias de los sistemas SAP existentes.
2.  SQL Server 2012 CU4 y posterior puede realizar copias de seguridad de las bases de datos en una dirección URL de almacenamiento de Azure.
3.  Copias de seguridad de instantáneas de archivo para archivos de base de datos en Azure Blob Storage. Este método solo funciona si los archivos de registro y los datos de SQL Server se encuentran en Azure Blob Storage.

El primer método es muy conocido y también se aplica en muchos casos de ubicaciones locales. No obstante, le deja la tarea de resolver la ubicación de copia de seguridad a más largo plazo. Como no quiere conservar las copias de seguridad durante 30 o más días en el almacenamiento de Azure Storage conectado localmente, deberá usar los servicios de Azure Backup u otra herramienta de copia de seguridad o recuperación de terceros que incluya la administración de acceso y de retención de las copias de seguridad. La alternativa es crear en Azure un servidor de archivos de gran tamaño usando espacios de almacenamiento de Windows.

El segundo método se describe con más detalle en el artículo [Copia de seguridad en URL de SQL Server](/sql/relational-databases/backup-restore/sql-server-backup-to-url?view=sql-server-2017). Hay distintas versiones de SQL Server que presentan algunas variaciones respecto a esta funcionalidad. Por lo tanto, debe consultar la documentación de la comprobación de su versión de SQL Server concreta. Es importante tener en cuenta que en este artículo se detallan numerosas restricciones. Tiene la posibilidad de hacer la copia de seguridad en:

- Un blob en páginas de Azure, que limita el tamaño de copia de seguridad a 1000 GB. Esto también limita el rendimiento que se puede alcanzar.
- Varios blobs en bloques de Azure (hasta 64), que posibilitan un tamaño teórico de copia de seguridad de 12 TB. En cambio, se han hecho pruebas con bases de datos de clientes que revelan que el tamaño máximo de copia de seguridad puede ser menor que el límite teórico. En este caso, usted es responsable de administrar la retención de copias de seguridad, así como del acceso a las copias de seguridad.


### <a name="automated-backup-for-sql-server"></a>Copia de seguridad automatizada para SQL Server
Copia de seguridad automatizada proporciona un servicio de copia de seguridad automático para las ediciones de SQL Server Standard y Enterprise que se ejecutan en una VM de Windows en Azure. Este servicio se proporciona por la [extensión del agente de IaaS de SQL Server](../../../azure-sql/virtual-machines/windows/sql-server-iaas-agent-extension-automate-management.md), que se instala automáticamente en las imágenes de máquina virtual de Windows de SQL Server en Azure Portal. Si implementa sus propias imágenes de sistema operativo con SQL Server instalado, deberá instalar las extensiones de máquina virtual por separado. Los pasos necesarios están documentados en este [artículo](../../../azure-sql/virtual-machines/windows/sql-server-iaas-agent-extension-automate-management.md).

En estos artículos encontrará más información detallada sobre las capacidades de este método:

- SQL Server 2014: [Automated Backup para SQL Server 2014 en Azure Virtual Machines (Resource Manager)](../../../azure-sql/virtual-machines/windows/automated-backup-sql-2014.md)
- SQL Server 2016/2017: [Automated Backup v2 para Azure Virtual Machines (Resource Manager)](../../../azure-sql/virtual-machines/windows/automated-backup.md)

Si consulta la documentación, verá que se ha mejorado la funcionalidad con las versiones más recientes de SQL Server. En el artículo [Copia de seguridad administrada de SQL Server en Microsoft Azure](/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure?view=sql-server-2017) se publica más información detallada sobre las copias de seguridad automatizadas de SQL Server. El límite de tamaño teórico de copia de seguridad es de 12 TB.  Las copias de seguridad automatizadas pueden ser un buen método para las copias de seguridad de hasta 12 TB. Dado que se escriben varios blobs en paralelo, puede prever un rendimiento de más de 100 MB/s. 
 

### <a name="azure-backup-for-sql-server-vms"></a>Azure Backup para VM con SQL Server
Este nuevo método de copias de seguridad de SQL Server está disponible desde junio de 2018 como una versión preliminar pública en los servicios de Azure Backup. El método de copia de seguridad de SQL Server es el mismo que usan otras herramientas de terceros (la interfaz VSS/VDI de SQL Server) para transmitir copias de seguridad a una ubicación de destino. En este caso, la ubicación de destino es el almacén de Azure Recovery Services.

[Aquí](../../../backup/backup-azure-sql-database.md) encontrará una descripción más detallada de este método de copia de seguridad, que presenta numerosas ventajas de configuraciones de copia de seguridad central, supervisión y administración. 


### <a name="third-party-backup-solutions"></a>Soluciones de copia de seguridad de terceros
Para no pocos clientes de SAP no se podía volver a empezar e introducir nuevas soluciones de copia de seguridad completas para la parte de su entorno de SAP que se ejecutaba en Azure. Como resultado, las soluciones de copia de seguridad existentes se tenían que usar y ampliar en Azure. La extensión de las soluciones de copia de seguridad existentes a Azure solía funcionar bien con la mayoría de los proveedores principales en este espacio. 


## <a name="using-a-sql-server-image-out-of-the-microsoft-azure-marketplace"></a><a name="1b353e38-21b3-4310-aeb6-a77e7c8e81c8"></a>Uso de imágenes de SQL Server desde Microsoft Azure Marketplace
Microsoft ofrece en Azure Marketplace máquinas virtuales que ya contienen versiones de SQL Server. Para los clientes de SAP que requieran licencias de SQL Server y Windows, el uso de estas imágenes podría ser una oportunidad para cubrir la necesidad de licencias activando las máquinas virtuales con SQL Server ya instalado. Para poder utilizar dichas imágenes para SAP, deben tenerse en cuenta las siguientes consideraciones:

* Las versiones de SQL Server que no son de evaluación incurren en costos más elevados que una máquina virtual exclusivamente con Windows implementada desde Azure Marketplace. Consulte estos artículos para comparar los precios: <https://azure.microsoft.com/pricing/details/virtual-machines/windows/> y <https://azure.microsoft.com/pricing/details/virtual-machines/sql-server-enterprise/>. 
* Solo se pueden usar las versiones de SQL Server compatibles con SAP.
* La intercalación de la instancia de SQL Server que está instalada en las máquinas virtuales que se ofrecen en Azure Marketplace no es la que requiere SAP NetWeaver para ejecutar la instancia de SQL Server. Puede cambiar la intercalación, aunque debe seguir las instrucciones de la sección que encontrará a continuación.

### <a name="changing-the-sql-server-collation-of-a-microsoft-windowssql-server-vm"></a>Modificación de la intercalación de SQL Server de una máquina virtual de Windows o SQL Server
Puesto que las imágenes de SQL Server en Azure Marketplace no están configuradas para usar la intercalación necesaria para las aplicaciones de SAP NetWeaver, debe modificarse inmediatamente después de llevar a cabo la implementación. Para SQL Server, este cambio de intercalación se puede efectuar siguiendo estos pasos en cuanto se haya implementado la máquina virtual y un administrador pueda iniciar sesión en la máquina virtual implementada:

* Abra una ventana de comandos de Windows como administrador.
* Cambie el directorio a C:\Archivos de programa\Microsoft SQL Server\110\Setup Bootstrap\SQLServer2012.
* Ejecute el comando: Setup.exe /QUIET /ACTION=REBUILDDATABASE /INSTANCENAME=MSSQLSERVER /SQLSYSADMINACCOUNTS=`<local_admin_account_name`> /SQLCOLLATION=SQL_Latin1_General_Cp850_BIN2.   
  * `<local_admin_account_name`&gt; es la cuenta que se definió como cuenta de administrador al implementar la máquina virtual por primera vez por medio de la galería.

Este proceso solo debe llevar unos minutos. Para asegurarse de que obtiene el resultado correcto, realice los pasos siguientes:

* Abra SQL Server Management Studio.
* Abra una ventana de consulta.
* Ejecute el comando sp_helpsort en la base de datos maestra de SQL Server.

El resultado deberá ser similar al que se muestra a continuación:

```output
Latin1-General, binary code point comparison sort for Unicode Data, SQL Server Sort Order 40 on Code Page 850 for non-Unicode Data
```

Si el resultado es diferente, DETENGA la implementación de SAP e investigue por qué el comando de instalación no funcionó como se esperaba. **NO** se admite la implementación de aplicaciones de SAP NetWeaver en la instancia de SQL Server con páginas de códigos de SQL Server distintas de la mencionada antes.

## <a name="sql-server-high-availability-for-sap-in-azure"></a>Alta disponibilidad de SQL Server para SAP en Azure
Si usa SQL Server en implementaciones de IaaS de Azure para SAP, dispondrá de distintas posibilidades para implementar la capa de DBMS de alta disponibilidad. Como ya se ha descrito en [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md), Azure proporciona distintos acuerdos de nivel de servicio de tiempo de actividad para una máquina virtual y un par de máquinas virtuales implementadas en un conjunto de disponibilidad de Azure. Se supone que tiene como objetivo el acuerdo de nivel de servicio de tiempo de actividad para sus implementaciones de producción que requieren la implementación en conjuntos de disponibilidad de Azure. En tal caso, deberá implementar un mínimo de dos máquinas virtuales en dicho conjunto de disponibilidad. Una máquina virtual ejecutará la instancia activa de SQL Server. La otra máquina virtual ejecutará la instancia pasiva

### <a name="sql-server-clustering-using-windows-scale-out-file-server"></a>Clústeres de SQL Server mediante el servidor de archivos de escalabilidad horizontal de Windows
Con Windows Server 2016, Microsoft introdujo los [espacios de almacenamiento directo](/windows-server/storage/storage-spaces/storage-spaces-direct-overview). En función de la implementación de Espacios de almacenamiento directo, se admite la agrupación en clústeres de FCI de SQL Server. Encontrará información detallada en el artículo [Configuración de una instancia de clúster de conmutación por error de SQL Server en Azure Virtual Machines](../../../azure-sql/virtual-machines/windows/failover-cluster-instance-storage-spaces-direct-manually-configure.md). La solución también requiere un equilibrador de carga de Azure para tratar la dirección IP virtual de los recursos de clúster. Los archivos de base de datos de SQL Server se almacenan en espacios de almacenamiento. Por lo tanto, es un hecho que se le pedirá que cree los espacios de almacenamiento de Windows a partir de Azure Premium Storage. Dado que esta solución se admite desde no hace mucho, no hay ningún cliente conocido de SAP que use esta solución en los escenarios de producción de SAP.  

### <a name="sql-server-log-shipping"></a>Trasvase de registros de SQL Server
Uno de los métodos de alta disponibilidad (HA) consiste en trasvasar los registros de SQL Server. Si las máquinas virtuales que participan en la configuración de alta disponibilidad tienen una resolución de nombres correcta, no habrá ningún problema y la configuración de Azure no es diferente a cualquier otra realizada de forma local. En lo que respecta a la configuración de trasvase de registros y sus principios. En el artículo [Acerca del trasvase de registros (SQL Server)](/sql/database-engine/log-shipping/about-log-shipping-sql-server?view=sql-server-2017) encontrará información detallada sobre el trasvase de registros de SQL Server.

La funcionalidad de trasvase de registros de SQL Server apenas se usó en Azure para lograr una alta disponibilidad dentro de una región de Azure. En cambio, en los siguientes escenarios, los clientes de SAP usaban el trasvase de registros adecuadamente junto con Azure:

- Escenarios de recuperación ante desastres de una región de Azure a otra
- Configuración de la recuperación ante desastres de una ubicación local a una región de Azure
- Escenarios de migración de una ubicación local a Azure. En esos casos, se usa el trasvase de registros para sincronizar la nueva implementación de DBMS en Azure con el sistema de producción actual a nivel local. En el momento de la migración, el sistema de producción se apaga y se garantiza que las copias de seguridad del registro de transacciones más recientes se hayan transferido a la implementación de DBMS de Azure. Después, la implementación de DBMS de Azure se abre para producción.  



### <a name="database-mirroring"></a>Creación de reflejo de la base de datos
La funcionalidad de creación de reflejo de la base de datos, que es compatible con SAP (consulte la nota de SAP [965908]), se basa en la definición de un asociado de conmutación por error en la cadena de conexión de SAP. Para los casos entre locales, asumimos que las dos máquinas virtuales están en el mismo dominio y que las dos instancias de SQL Server se están ejecutando como un usuario del dominio, así como que tendrán los privilegios suficientes en las dos instancias de SQL Server. Por lo tanto, el proceso configuración de la creación de reflejo de la base de datos de Azure es el mismo en una configuración o instalación local típica.

Al igual que con las implementaciones exclusivas en la nube, el método más fácil es tener otra configuración de dominio en Azure con el fin de albergar esas máquinas virtuales de DBMS (e, idealmente, las máquinas virtuales de SAP específicas) dentro de un dominio.

Si un dominio no es posible, también se pueden usar certificados para los puntos de conexión de reflejo de la base de datos, como se describe aquí: <https://docs.microsoft.com/sql/database-engine/database-mirroring/use-certificates-for-a-database-mirroring-endpoint-transact-sql>

Aquí encontrará un tutorial para configurar la creación de reflejo de la base de datos en Azure: <https://docs.microsoft.com/sql/database-engine/database-mirroring/database-mirroring-sql-server> 

### <a name="sql-server-always-on"></a>SQL Server AlwaysOn
Como AlwaysOn es compatible con un entorno de SAP local (consulte la nota de SAP [1772688]), se puede usar en combinación con SAP en Azure. Hay algunas consideraciones especiales respecto a implementar el agente de escucha de grupo de disponibilidad de SQL Server (no debe confundirse con el conjunto de disponibilidad de Azure), ya que Azure en este momento no permite crear un objeto AD o DNS como en las implementaciones locales. Por lo tanto, hay que realizar otros pasos de instalación para solucionar el comportamiento específico de Azure.

Estas son algunas de las consideraciones que hay que tener en cuenta al usar un agente de escucha de grupo de disponibilidad:

* Solo se puede usar un agente de escucha de grupo de disponibilidad con Windows Server 2012 o posterior como SO invitado de la máquina virtual. Para Windows Server 2012, debe asegurarse de que se aplica esta revisión: <https://support.microsoft.com/kb/2854082> 
* Esta revisión no está disponible para Windows Server 2008 R2 y AlwaysOn tendría que usarse de la misma manera que la funcionalidad de creación de reflejo de base de datos mediante la especificación de un asociado de conmutación por error en la cadena de conexiones (se realiza mediante el parámetro de SAP default.pfl dbs/mss/server; consulte la nota de SAP [965908]).
* Cuando se utiliza un agente de escucha de grupo de disponibilidad, las máquinas virtuales de la base de datos tienen que estar conectadas a un equilibrador de carga específico. Para evitar que Azure asigne nuevas direcciones IP en casos donde ambas máquinas virtuales se apaguen accidentalmente, se deben asignar direcciones IP estáticas a las interfaces de red de esas máquinas virtuales en la configuración de AlwaysOn (la definición de una dirección IP estática se describe en [este][virtual-networks-reserved-private-ip] artículo)
* Hay que realizar algunos pasos especiales al crear la configuración del clúster WSFC: el clúster necesita una dirección IP especial, ya la funcionalidad actual de Azure asignaría el nombre del clúster a la misma dirección IP que el nodo donde se ha creado dicho clúster. Es decir, se debe realizar un paso manual para asignar una dirección IP diferente a la del clúster.
* El agente de escucha de grupo de disponibilidad se va a crear en Azure con puntos de conexión TCP/IP asignados a las máquinas virtuales que ejecutan las réplicas principal y secundarias del grupo de disponibilidad.
* Puede que haya que proteger estos puntos de conexión con ACL.

A continuación encontrará documentación detallada sobre cómo implementar AlwaysOn con SQL Server en máquinas virtuales de Azure:

- [Introducción a los grupos de disponibilidad Always On de SQL Server en Azure Virtual Machines](../../../azure-sql/virtual-machines/windows/availability-group-overview.md).
- [Configuración de un grupo de disponibilidad AlwaysOn en máquinas virtuales de Azure en distintas regiones](../../../azure-sql/virtual-machines/windows/availability-group-manually-configure-multiple-regions.md).
- [Configuración de un equilibrador de carga para un grupo de disponibilidad AlwaysOn en Azure](../../../azure-sql/virtual-machines/windows/availability-group-load-balancer-portal-configure.md).

>[!NOTE]
> Si va a configurar el equilibrador de carga de Azure para la dirección IP virtual del agente de escucha de grupo de disponibilidad, asegúrese de que DirectServerReturn esté configurado. La configuración de esta opción reducirá la latencia de recorrido de ida y vuelta de red entre la capa de aplicación de SAP y la capa de DBMS. 

SQL Server AlwaysOn es la funcionalidad de alta disponibilidad y de recuperación ante desastres de uso más común en Azure para las implementaciones de carga de trabajo de SAP. La mayoría de los clientes usan AlwaysOn para la alta disponibilidad en una única región de Azure. Si la implementación está restringida a solo dos nodos, tiene dos opciones de conectividad:

- Usar el agente de escucha de grupo de disponibilidad. Con el agente de escucha del grupo de disponibilidad, deberá implementar un equilibrador de carga de Azure. Este suele ser el método predeterminado de implementación. Las aplicaciones de SAP se configurarían para conectarse en el agente de escucha del grupo de disponibilidad, y no en un único nodo.
- Usar los parámetros de conectividad de creación de reflejo de la base de datos de SQL Server. En este caso, deberá configurar la conectividad de las aplicaciones de SAP de manera que se denominen ambos nombres de nodo. En la nota de SAP [#965908](https://launchpad.support.sap.com/#/notes/965908) se documenta la información exacta de dicha configuración de SAP. Con esta opción no tendría que configurar ningún agente de escucha del grupo de disponibilidad y, con eso, ningún equilibrador de carga de Azure para la alta disponibilidad de SQL Server. Como resultado, la latencia de red entre la capa de aplicación de SAP y la capa de DBMS es menor, ya que el tráfico de entrada a la instancia de SQL Server no se enruta a través del equilibrador de carga de Azure. Pero debe recordar que esta opción solo funciona si restringe el grupo de disponibilidad para que abarque dos instancias. 

Unos pocos clientes están aprovechando la funcionalidad de SQL Server AlwaysOn de una funcionalidad adicional de recuperación ante desastres entre las regiones de Azure. Muchos clientes también usan la capacidad de realizar copias de seguridad desde una réplica secundaria. 

## <a name="sql-server-transparent-data-encryption"></a>Cifrado de datos transparente de SQL Server
Hay una serie de clientes que usan el [Cifrado de datos transparente (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption?view=sql-server-2017) de SQL Server al implementar sus bases de datos de SQL Server de SAP en Azure. La funcionalidad de TDE de SQL Server es totalmente compatible con SAP (consulte la nota de SAP [1380493 #](https://launchpad.support.sap.com/#/notes/1380493)). 

### <a name="applying-sql-server-tde"></a>Aplicar el TDE de SQL Server
En los casos en que se efectúe una migración heterogénea desde otro DBMS, que se ejecuta en un entorno local, a Windows o SQL Server, que se ejecuta en Azure, se debería crear de antemano una base de datos de destino vacía en SQL Server. El siguiente paso sería aplicaría la funcionalidad de TDE de SQL Server mientras se sigue ejecutando el sistema de producción de forma local. El motivo de seguir esta secuencia es que el proceso de cifrado de la base de datos vacía puede tardar bastante. Luego, los procesos de importación SAP importarían los datos a la base de datos cifrada durante la fase de tiempo de inactividad. La sobrecarga de la importación a una base de datos cifrada tiene un impacto temporal mucho menor de manera que el cifrado de la base de datos después de la fase de exportación en la fase de tiempo de inactividad. Se han dado experiencias negativas al intentar aplicar TDE con la carga de trabajo de SAP ejecutada en la base de datos. Por lo tanto, se recomienda tratar la implementación de TDE como una actividad que debe llevarse a cabo sin ninguna carga de trabajo de SAP en la base de datos concreta.

En casos en los que se mueven bases de datos de SQL Server de SAP desde una ubicación local hasta Azure, se recomienda probar en qué infraestructura se puede aplicar con mayor rapidez el cifrado. Respecto a esto, debe tener en cuenta estos factores:

- No puede definir la cantidad de subprocesos que se usan para aplicar el cifrado de datos a la base de datos. El número de subprocesos depende principalmente del número de volúmenes de disco por los que se distribuyen los datos y los archivos de registro de SQL Server. Significa que, cuantos más diferentes sean los volúmenes (letras de unidad), más subprocesos se ocuparán en paralelo para realizar el cifrado. Esta configuración se contradice un poco con una sugerencia anterior sobre la configuración de disco según la cual se crea uno o unos pocos espacios de almacenamiento para los archivos de base de datos de SQL Server en las máquinas virtuales de Azure. Una configuración con pocos volúmenes daría lugar a pocos subprocesos que ejecutan el cifrado. Un cifrado de un único subproceso lee extensiones de 64 KB, las cifra y, después, escribe un registro en el archivo de registro de transacciones indicando que se ha cifrado la extensión. Como resultado, la carga en el registro de transacciones es moderada.
- En versiones anteriores de SQL Server, la compresión de copia de seguridad ya no implicaba eficacia alguna al cifrar la base de datos de SQL Server. Este comportamiento podría convertirse en un problema si su idea era cifrar la base de datos de SQL Server de forma local y, después, copiar una copia de seguridad en Azure para restaurar la base de datos en Azure. La compresión de copia de seguridad de SQL Server suele generar una razón de compresión de factor 4.
- Con SQL Server 2016, SQL Server introdujo una nueva funcionalidad que también permite comprimir bases de datos cifradas de una forma eficiente. Consulte [este blog](/archive/blogs/sqlcat/sqlsweet16-episode-1-backup-compression-for-tde-enabled-databases) para obtener información detallada.
 
Si solo trata la aplicación del cifrado de TDE con ninguna carga de trabajo de SAP, o con una ínfima, debería efectuar pruebas en su configuración específica sobre si es mejor aplicar el TDE a la base de datos de SAP local o aplicarlo a Azure. En Azure sin duda tiene más flexibilidad en términos de exceso de aprovisionamiento de infraestructura y de reducir la infraestructura una vez aplicado el TDE.

### <a name="using-azure-key-vault"></a>Uso de Azure Key Vault
Azure ofrece el servicio de un [almacén de claves](https://azure.microsoft.com/services/key-vault/) para almacenar claves de cifrado. Por otro lado, SQL Server ofrece un conector para aprovechar Azure Key Vault como almacén para los certificados de TDE.

A continuación tiene más información detallada sobre cómo usar Azure Key Vault para el TDE de SQL Server:

- [Administración extensible de claves mediante Azure Key Vault (SQL Server)](/sql/relational-databases/security/encryption/extensible-key-management-using-azure-key-vault-sql-server?view=sql-server-2017).
- [Pasos de configuración de Administración extensible de claves de TDE de SQL Server mediante Azure Key Vault](/sql/relational-databases/security/encryption/setup-steps-for-extensible-key-management-using-the-azure-key-vault?view=sql-server-2017).
- [Mantenimiento y solución de problemas del Conector de SQL Server](/sql/relational-databases/security/encryption/sql-server-connector-maintenance-troubleshooting?view=sql-server-2017).
- [Más preguntas de los clientes sobre el cifrado de datos transparente de SQL Server – TDE + Azure Key Vault](/archive/blogs/saponsqlserver/more-questions-from-customers-about-sql-server-transparent-data-encryption-tde-azure-key-vault).


>[!IMPORTANT]
>Con TDE de SQL Server, sobre todo con Azure Key Vault, se recomienda usar las revisiones más recientes de SQL Server 2014, SQL Server 2016 y SQL Server 2017. El motivo es que, a partir de los comentarios de los clientes, se aplicaron optimizaciones y correcciones al código. A modo de ejemplo, compruebe [KBA #4058175](https://support.microsoft.com/help/4058175/tde-enabled-backup-and-restore-slow-if-encryption-key-is-stored-in-ekm).
>  

## <a name="general-sql-server-for-sap-on-azure-summary"></a><a name="9053f720-6f3b-4483-904d-15dc54141e30"></a>Resumen general de SQL Server para SAP en Azure
En esta guía se ofrecen muchas recomendaciones: le sugerimos que la lea más de una vez antes de planear la implementación de Azure. En general, asegúrese de seguir las principales recomendaciones específicas de DBMS en Azure:

1. Use la versión de DBMS más reciente, como SQL Server 2017, que presenta la mayoría de las ventajas de Azure. 
2. Planee cuidadosamente su infraestructura de sistema SAP en Azure para equilibrar el diseño del archivo de datos y las restricciones de Azure:
   * No disponga de un número de discos demasiado elevado, solo el suficiente para asegurarse de que puede alcanzar los IOPS necesarios.
   * Si no utiliza Managed Disks, recuerde que los valores de IOPS también están limitados por cuenta de Azure Storage y que las cuentas de Storage están limitadas en cada suscripción de Azure ([más información][azure-resource-manager/management/azure-subscription-service-limits]). 
   * Cree secciones en los discos solo si necesita obtener un mayor rendimiento.
3. Nunca instale software o coloque los archivos que requieren persistencia en la unidad D:\, ya que no es permanente y todos los datos que albergue se pierden tras reiniciar Windows.
4. No utilice el almacenamiento en caché de disco para Azure Standard Storage.
5. No use cuentas de Azure Standard Storage con replicación geográfica de Azure.  Use la redundancia local para las cargas de trabajo de DBMS.
6. Utilice la solución de alta disponibilidad o recuperación ante desastres de su proveedor de DBMS para replicar datos de base de datos.
7. Use siempre la resolución de nombres, no confíe exclusivamente en las direcciones IP.
8. Con el TDE de SQL Server, aplique las revisiones más recientes de SQL Server.
9. Use el nivel de compresión de base de datos más elevado posible. Qué es la comprensión de página de SQL Server.
10. Tenga cuidado al utilizar imágenes de SQL Server de Azure Marketplace. Si lo hace, debe cambiar la intercalación de la instancia antes de instalar cualquier sistema SAP NetWeaver en ella.
11. Instale y configure la funcionalidad de supervisión de hosts de SAP para Azure tal y como se describe en la [Guía de implementación][deployment-guide].