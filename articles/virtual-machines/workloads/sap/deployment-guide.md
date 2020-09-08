---
title: Implementación de Azure Virtual Machines para SAP NetWeaver | Microsoft Docs
description: Aprenda a implementar software de SAP en máquinas virtuales Linux en Azure.
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: MSSedusch
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 1c4f1951-3613-4a5a-a0af-36b85750c84e
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/16/2020
ms.author: sedusch
ms.openlocfilehash: 6b66db639e0b22a7e0c45486465a1fa395aba33d
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88653518"
---
# <a name="azure-virtual-machines-deployment-for-sap-netweaver"></a>Implementación de Azure Virtual Machines para SAP NetWeaver

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
[2367194]:https://launchpad.support.sap.com/#/notes/2367194

[azure-cli]:../../../cli-install-nodejs.md
[azure-cli-2]:https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-portal]:https://portal.azure.com
[azure-ps]:/powershell/azure/
[azure-quickstart-templates-github]:https://github.com/Azure/azure-quickstart-templates
[azure-script-ps]:https://go.microsoft.com/fwlink/p/?LinkID=395017
[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md#subscription-limits

[dbms-guide]:dbms-guide.md (Implementación de DBMS de Azure Virtual Machines para SAP)
[dbms-guide-2.1]:dbms-guide.md#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f (Almacenamiento en caché de máquinas virtuales y VHD)
[dbms-guide-2.2]:dbms-guide.md#c8e566f9-21b7-4457-9f7f-126036971a91 (Software RAID)
[dbms-guide-2.3]:dbms-guide.md#10b041ef-c177-498a-93ed-44b3441ab152 (Microsoft Azure Storage)
[dbms-guide-2]:dbms-guide.md#65fa79d6-a85f-47ee-890b-22e794f51a64 (Estructura de una implementación de RDBMS)
[dbms-guide-3]:dbms-guide.md#871dfc27-e509-4222-9370-ab1de77021c3 (Alta disponibilidad y recuperación ante desastres con máquinas virtuales de Azure)
[dbms-guide-5.5.1]:dbms-guide.md#0fef0e79-d3fe-4ae2-85af-73666a6f7268 (SQL Server 2012 SP1 CU4 y versiones posteriores)
[dbms-guide-5.5.2]:dbms-guide.md#f9071eff-9d72-4f47-9da4-1852d782087b (SQL Server 2012 SP1 CU3 y versiones anteriores)
[dbms-guide-5.6]:dbms-guide.md#1b353e38-21b3-4310-aeb6-a77e7c8e81c8 (Uso de una imagen de SQL Server desde Azure Marketplace)
[dbms-guide-5.8]:dbms-guide.md#9053f720-6f3b-4483-904d-15dc54141e30 (Resumen general de SQL Server para SAP en Azure)
[dbms-guide-5]:dbms-guide.md#3264829e-075e-4d25-966e-a49dad878737 (Especificaciones de RDBMS de SQL Server)
[dbms-guide-8.4.1]:dbms-guide.md#b48cfe3b-48e9-4f5b-a783-1d29155bd573 (Configuración de almacenamiento)
[dbms-guide-8.4.2]:dbms-guide.md#23c78d3b-ca5a-4e72-8a24-645d141a3f5d (Copia de seguridad y restauración)
[dbms-guide-8.4.3]:dbms-guide.md#77cd2fbb-307e-4cbf-a65f-745553f72d2c (Consideraciones de rendimiento para copias de seguridad y restauraciones)
[dbms-guide-8.4.4]:dbms-guide.md#f77c1436-9ad8-44fb-a331-8671342de818 (Otros)
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

[deployment-guide]:deployment-guide.md (Implementación de Azure Virtual Machines para SAP)
[deployment-guide-2.2]:deployment-guide.md#42ee2bdb-1efc-4ec7-ab31-fe4c22769b94 (Recursos SAP)
[deployment-guide-3.1.2]:deployment-guide.md#3688666f-281f-425b-a312-a77e7db2dfab (Implementación de una máquina virtual mediante una imagen personalizada)
[deployment-guide-3.2]:deployment-guide.md#db477013-9060-4602-9ad4-b0316f8bb281 (Escenario 1: Implementación de una máquina virtual desde Azure Marketplace para SAP)
[deployment-guide-3.3]:deployment-guide.md#54a1fc6d-24fd-4feb-9c57-ac588a55dff2 (Escenario 2: Implementación de una máquina virtual con una imagen personalizada para SAP)
[deployment-guide-3.4]:deployment-guide.md#a9a60133-a763-4de8-8986-ac0fa33aa8c1 (Escenario 3: Traslado de una máquina virtual desde una ubicación local con un VHD no generalizado de Azure con SAP)
[deployment-guide-3]:deployment-guide.md#b3253ee3-d63b-4d74-a49b-185e76c4088e (Escenarios de implementación de máquinas virtuales para SAP en Microsoft Azure)
[deployment-guide-4.1]:deployment-guide.md#604bcec2-8b6e-48d2-a944-61b0f5dee2f7 (Implementación de cmdlets de Azure PowerShell)
[deployment-guide-4.2]:deployment-guide.md#7ccf6c3e-97ae-4a7a-9c75-e82c37beb18e (Descarga e importación de cmdlets de PowerShell significativos para SAP)
[deployment-guide-4.3]:deployment-guide.md#31d9ecd6-b136-4c73-b61e-da4a29bbc9cc (Unión de una máquina virtual a un dominio local - solo Windows)
[deployment-guide-4.4.2]:deployment-guide.md#6889ff12-eaaf-4f3c-97e1-7c9edc7f7542 (Linux)
[deployment-guide-4.4]:deployment-guide.md#c7cbb0dc-52a4-49db-8e03-83e7edc2927d (Descarga, instalación y habilitación del agente de máquina virtual de Azure)
[deployment-guide-4.5.1]:deployment-guide.md#987cf279-d713-4b4c-8143-6b11589bb9d4 (Azure PowerShell)
[deployment-guide-4.5.2]:deployment-guide.md#408f3779-f422-4413-82f8-c57a23b4fc2f (CLI de Azure)
[deployment-guide-4.5]:deployment-guide.md#d98edcd3-f2a1-49f7-b26a-07448ceb60ca (Configuración de la extensión de Azure para SAP)
[deployment-guide-configure-new-extension-ps]:deployment-guide.md#2ad55a0d-9937-4943-9dd2-69bc2b5d3de0 (Configuración de la nueva extensión de Azure para SAP con Azure PowerShell)
[deployment-guide-configure-new-extension-cli]:deployment-guide.md#c8749c24-fada-42ad-b114-f9aae2dc37da (Configuración de la nueva extensión de Azure para SAP con la CLI de Azure)
[deployment-guide-5.1]:deployment-guide.md#bb61ce92-8c5c-461f-8c53-39f5e5ed91f2 (Comprobación de la preparación de la extensión de Azure para SAP)
[deployment-guide-5.1-new]:deployment-guide.md#7bf24f59-7347-4c7a-b094-4693e4687ee5 (Comprobación de la preparación de la nueva extensión de Azure para SAP)
[deployment-guide-5.2]:deployment-guide.md#e2d592ff-b4ea-4a53-a91a-e5521edb6cd1 (Comprobación del estado de la configuración de la extensión de Azure para SAP)
[deployment-guide-5.2-new]:deployment-guide.md#464ac96d-7d3c-435d-a5ae-3faf3bfef4b3 (Comprobación del estado de la configuración de la nueva extensión de Azure para SAP)
[deployment-guide-5.3]:deployment-guide.md#fe25a7da-4e4e-4388-8907-8abc2d33cfd8 (Solución de problemas de la extensión de Azure para SAP)
[deployment-guide-5.3-new]:deployment-guide.md#b7afb8ef-a64c-495d-bb37-2af96688c530 (Solución de problemas de la nueva extensión de Azure para SAP)
[deployment-guide-contact-support]:deployment-guide.md#3ba34cfc-c9bb-4648-9c3c-88e8b9130ca2 (Solución de problemas de la extensión de Azure para SAP: póngase en contacto con el soporte técnico)
[deployment-guide-run-the-script]:deployment-guide.md#0d2847ad-865d-4a4c-a405-f9b7baaa00c7 (Solución de problemas de la extensión de Azure para SAP: ejecute el script de configuración)
[deployment-guide-redeploy-after-sysprep]:deployment-guide.md#2cd61f22-187d-42ed-bb8c-def0c983d756 (Solución de problemas de la extensión de Azure para SAP: repita la implementación después de sysprep)
[deployment-guide-fix-internet-connection]:deployment-guide.md#e92bc57d-80d9-4a2b-a2f4-16713a22ad89 (Solución de problemas de la extensión de Azure para SAP: corrija los problemas de conexión a Internet)


[deployment-guide-configure-monitoring-scenario-1]:deployment-guide.md#ec323ac3-1de9-4c3a-b770-4ff701def65b (Configuración de la extensión de máquina virtual)
[deployment-guide-configure-proxy]:deployment-guide.md#baccae00-6f79-4307-ade4-40292ce4e02d (Configuración de proxy)
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
[deployment-guide-troubleshooting-chapter]:deployment-guide.md#564adb4f-5c95-4041-9616-6635e83a810b (Comprobaciones y solución de problemas)

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

[msdn-set-Azvmaemextension]:https://docs.microsoft.com/powershell/module/az.compute/set-azvmaemextension

[planning-guide]:planning-guide.md (Planeamiento e implementación de Azure Virtual Machines para SAP)
[planning-guide-1.2]:planning-guide.md#e55d1e22-c2c8-460b-9897-64622a34fdff (Recursos)
[planning-guide-11]:planning-guide.md#7cf991a1-badd-40a9-944e-7baae842a058 (Alta disponibilidad y recuperación ante desastres para la ejecución de SAP NetWeaver en Azure Virtual Machines)
[planning-guide-11.4.1]:planning-guide.md#5d9d36f9-9058-435d-8367-5ad05f00de77 (Alta disponibilidad en los servidores de aplicaciones de SAP)
[planning-guide-11.5]:planning-guide.md#4e165b58-74ca-474f-a7f4-5e695a93204f (Uso del inicio automático en las instancias de SAP)
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803 (Solo en la nube: implementaciones de máquina virtual en Azure sin dependencias en la red local del cliente)
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10 (Entre locales: implementación de una o varias máquinas virtuales de SAP en Azure completamente integradas con la red local)
[planning-guide-3.1]:planning-guide.md#be80d1b9-a463-4845-bd35-f4cebdb5424a (Regiones de Azure)
[planning-guide-3.2.1]:planning-guide.md#df49dc09-141b-4f34-a4a2-990913b30358 (Dominios de error)
[planning-guide-3.2.2]:planning-guide.md#fc1ac8b2-e54a-487c-8581-d3cc6625e560 (Dominios de actualización)
[planning-guide-3.2.3]:planning-guide.md#18810088-f9be-4c97-958a-27996255c665 (Conjuntos de disponibilidad de Azure)
[planning-guide-3.2]:planning-guide.md#8d8ad4b8-6093-4b91-ac36-ea56d80dbf77 (Concepto de máquinas virtuales de Microsoft Azure)
[planning-guide-5.1.1]:planning-guide.md#4d175f1b-7353-4137-9d2f-817683c26e53 (Procedimiento para mover máquinas virtuales del entorno local a Azure con un disco no generalizado)
[planning-guide-5.1.2]:planning-guide.md#e18f7839-c0e2-4385-b1e6-4538453a285c (Implementación de una máquina virtual con una imagen específica del cliente)
[planning-guide-5.2.1]:planning-guide.md#1b287330-944b-495d-9ea7-94b83aff73ef (Preparación para mover máquinas virtuales del entorno local a Azure con un disco no generalizado)
[planning-guide-5.2.2]:planning-guide.md#57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3 (Preparación para la implementación de una máquina virtual con una imagen específica del cliente para SAP)
[planning-guide-5.2]:planning-guide.md#6ffb9f41-a292-40bf-9e70-8204448559e7 (Preparación de máquinas virtuales con SAP para Azure)
[planning-guide-5.3.1]:planning-guide.md#6e835de8-40b1-4b71-9f18-d45b20959b79 (Diferencia entre un disco de Azure y una imagen de Azure)
[planning-guide-5.3.2]:planning-guide.md#a43e40e6-1acc-4633-9816-8f095d5a7b6a (Carga de un VHD desde una instalación local a Azure)
[planning-guide-5.4.2]:planning-guide.md#9789b076-2011-4afa-b2fe-b07a8aba58a1 (Copia de discos entre cuentas de Azure Storage)
[planning-guide-5.5.1]:planning-guide.md#4efec401-91e0-40c0-8e64-f2dceadff646 (Estructura de VM/VHD para implementaciones de SAP)
[planning-guide-5.5.3]:planning-guide.md#17e0d543-7e8c-4160-a7da-dd7117a1ad9d (Configuración de montaje automático para discos conectados)
[planning-guide-9.1]:planning-guide.md#6f0a47f3-a289-4090-a053-2521618a28c3 (Solución de supervisión de Azure para SAP)
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
[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd (Redes de Microsoft Azure)
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f (Almacenamiento: Microsoft Azure Storage y discos de datos)

[resource-group-authoring-templates]:../../../resource-group-authoring-templates.md
[resource-group-overview]:../../../azure-resource-manager/management/overview.md
[resource-groups-networking]:../../../networking/network-overview.md
[sap-pam]:https://support.sap.com/pam (Matriz de disponibilidad de productos SAP)
[sap-templates-2-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-2-tier-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-marketplace-image-md%2Fazuredeploy.json
[sap-templates-2-tier-os-disk]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-disk%2Fazuredeploy.json
[sap-templates-2-tier-os-disk-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-disk-md%2Fazuredeploy.json
[sap-templates-2-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-image%2Fazuredeploy.json
[sap-templates-2-tier-user-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-image-md%2Fazuredeploy.json
[sap-templates-3-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-3-tier-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-md%2Fazuredeploy.json
[sap-templates-3-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-user-image%2Fazuredeploy.json
[sap-templates-3-tier-user-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-user-image-md%2Fazuredeploy.json
[storage-azure-cli]:../../../storage/common/storage-azure-cli.md
[storage-azure-cli-copy-blobs]:../../../storage/common/storage-azure-cli.md#copy-blobs
[storage-introduction]:../../../storage/common/storage-introduction.md
[storage-powershell-guide-full-copy-vhd]:../../../storage/common/storage-powershell-guide-full.md#how-to-copy-blobs-from-one-storage-container-to-another
[storage-premium-storage-preview-portal]:../../disks-types.md
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
[virtual-machines-linux-cli-deploy-templates]:../../linux/cli-deploy-templates.md (Implementación y administración de máquinas virtuales con plantillas de Azure Resource Manager y la CLI de Azure)
[virtual-machines-deploy-rmtemplates-powershell]:../../virtual-machines-windows-ps-manage.md (Administración de máquinas virtuales con Azure Resource Manager y PowerShell)
[virtual-machines-windows-agent-user-guide]:../../extensions/agent-windows.md
[virtual-machines-linux-agent-user-guide]:../../extensions/agent-linux.md
[virtual-machines-linux-agent-user-guide-command-line-options]:../../extensions/agent-linux.md#command-line-options
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
[virtual-machines-manage-availability]:../../linux/manage-availability.md
[virtual-machines-ps-create-preconfigure-windows-resource-manager-vms]:../../windows/quick-create-powershell.md
[virtual-machines-sizes]:../../linux/sizes.md
[virtual-machines-windows-classic-ps-sql-alwayson-availability-groups]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups.md
[virtual-machines-windows-classic-ps-sql-int-listener]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener.md
[virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions]:../../../azure-sql/virtual-machines/windows/business-continuity-high-availability-disaster-recovery-hadr-overview.md
[virtual-machines-sql-server-infrastructure-services]:../../../azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md
[virtual-machines-sql-server-performance-best-practices]:../../../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md
[virtual-machines-upload-image-windows-resource-manager]:../../windows/upload-image.md
[virtual-machines-windows-tutorial]:../../windows/quick-create-portal.md
[virtual-machines-workload-template-sql-alwayson]:https://azure.microsoft.com/documentation/templates/sql-server-2014-alwayson-dsc/
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
[qs-configure-powershell-windows-vm]:../../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md
[qs-configure-cli-windows-vm]:../../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md
[howto-assign-access-powershell]:../../../active-directory/managed-identities-azure-resources/howto-assign-access-powershell.md
[howto-assign-access-cli]:../../../active-directory/managed-identities-azure-resources/howto-assign-access-cli.md

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

Azure Virtual Machines es la solución para organizaciones que necesitan recursos de almacenamiento y proceso, en un tiempo mínimo y sin ciclos de adquisición prolongados. Puede usar Azure Virtual Machines para implementar en Azure aplicaciones clásicas, por ejemplo, aplicaciones basadas en SAP NetWeaver. Amplíe la confiabilidad y disponibilidad de una aplicación sin recursos locales adicionales. Azure Virtual Machines admite la conectividad entre locales, así que podrá integrar Azure Virtual Machines a los dominios locales, las nubes privadas y la infraestructura de sistema SAP de la organización.

En este artículo, trataremos los pasos a seguir para implementar aplicaciones de SAP en máquinas virtuales en Azure, incluidas las opciones de implementación alternativas y solución de problemas. Este artículo se basa en la información de [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver][planning-guide]. También complementa la documentación sobre la instalación de SAP y las notas de SAP, que son los recursos principales para la instalación e implementación de software de SAP.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

Configurar una máquina virtual de Azure para la implementación de software de SAP conlleva varios pasos y recursos. Antes de empezar, asegúrese de que cumple los requisitos previos para instalar el software SAP en máquinas virtuales en Azure.

### <a name="local-computer"></a>Equipo local

Para administrar máquinas virtuales Windows o Linux, puede usar un script de PowerShell y Azure Portal. Para ambas herramientas, necesita un equipo local que ejecute Windows 7 o una versión posterior de Windows. Si desea administrar solo máquinas virtuales Linux y desea usar un equipo Linux para esta tarea, puede utilizar la CLI de Azure.

### <a name="internet-connection"></a>Conexión a Internet

Para descargar y ejecutar las herramientas y los scripts necesarios para la implementación de software de SAP, debe estar conectado a Internet. Además, la máquina virtual de Azure que ejecuta la extensión de Azure para SAP debe tener acceso a Internet. Si la máquina virtual de Azure forma parte de una red virtual de Azure o de un dominio local, asegúrese de que la configuración de proxy correspondiente esté establecida, tal y como se describe en [Configuración de proxy][deployment-guide-configure-proxy].

### <a name="microsoft-azure-subscription"></a>Suscripción de Microsoft Azure

Necesita una cuenta de Azure activa.

### <a name="topology-and-networking"></a>Topología y redes

Debe definir la topología y la infraestructura de la implementación de SAP en Azure:

* Cuentas de Azure Storage que se utilizarán
* Red virtual en la que desea implementar el sistema SAP
* Grupo de recursos en el que desea implementar el sistema SAP
* Región de Azure en que desea implementar el sistema SAP
* Configuración de SAP (dos o tres niveles)
* Tamaños de máquina virtual y número de discos de datos adicionales que se montarán en las VM
* Configuración de SAP Correction and Transport System (CTS)

Cree y configure las cuentas de Azure Storage (si fuese necesario) o las redes virtuales de Azure antes de comenzar el proceso de implementación del software de SAP. Para más información acerca de cómo crear y configurar estos recursos, consulte [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver][planning-guide].

### <a name="sap-sizing"></a>Ajuste del tamaño de SAP

Obtenga la siguiente información para ajustar el tamaño de SAP:

* Calcule la carga de trabajo de SAP proyectada, por ejemplo, con la herramienta SAP Quick Sizer, y el valor del estándar de rendimiento de las aplicaciones SAP (SAPS)
* Cuánta memoria y cuántos recursos de CPU consume el sistema SAP
* Número de operaciones de entrada/salida (E/S) por segundo
* Ancho de banda de red que se requiere ante una eventual comunicación entre distintas máquinas virtuales en Azure
* Ancho de banda de red que se requiere entre los recursos locales y los sistemas SAP implementados en Azure

### <a name="resource-groups"></a>Grupos de recursos

En Azure Resource Manager, puede usar los grupos de recursos para administrar todos los recursos de aplicación de la suscripción de Azure. Para más información, consulte [Información general de Azure Resource Manager][resource-group-overview].

## <a name="resources"></a>Recursos

### <a name="sap-resources"></a><a name="42ee2bdb-1efc-4ec7-ab31-fe4c22769b94"></a>Recursos de SAP

Cuando se configura la implementación de software de SAP, se necesitan los siguientes recursos SAP:

* Nota de SAP [1928533], que incluye:
  * La lista de tamaños de máquina virtual de Azure que se admiten para la implementación del software de SAP
  * Información importante sobre la capacidad para los tamaños de máquina virtual de Azure
  * Software de SAP admitido y combinaciones de sistema operativo y base de datos
  * Versión del kernel de SAP requerida para Windows y Linux en Microsoft Azure

* La nota de SAP [2015553] enumera los requisitos previos para las implementaciones de software de SAP admitidas por SAP en Azure.
* La nota de SAP [2178632] contiene información detallada sobre todas las métricas de supervisión notificadas para SAP en Azure.
* La nota de SAP [1409604] incluye la versión de SAP Host Agent necesaria para Windows en Azure.
* La nota de SAP [2191498] incluye la versión de SAP Host Agent necesaria para Linux en Azure.
* La nota de SAP [2243692] incluye información acerca de las licencias de SAP en Linux en Azure.
* La nota de SAP [1984787] incluye información general sobre SUSE Linux Enterprise Server 12.
* La nota de SAP [2002167] incluye información general sobre Red Hat Enterprise Linux 7.x.
* La nota de SAP [2069760] incluye información general sobre Oracle Linux 7.x.
* La nota de SAP [1999351] contiene más soluciones de problemas de la extensión de Azure para SAP.
* La nota de SAP [1597355] incluye información general sobre el espacio de intercambio para Linux.
* La página de [SCN de SAP en Azure](https://wiki.scn.sap.com/wiki/x/Pia7Gg) tiene noticias y una colección de recursos útiles.
* La [WIKI de la comunidad SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contiene todas las notas de SAP que se necesitan para Linux.
* Cmdlets de PowerShell específicos de SAP que forman parte de [Azure PowerShell][azure-ps].
* Comandos de la CLI de Azure específicos de SAP que forman parte de la [CLI de Azure][azure-cli].

### <a name="windows-resources"></a><a name="42ee2bdb-1efc-4ec7-ab31-fe4c22769b94"></a>Recursos de Windows

Estos artículos de Microsoft se refieren a las implementaciones de SAP en Azure:

* [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver][planning-guide]
* [Implementación de Azure Virtual Machines para SAP NetWeaver (este artículo)][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP NetWeaver][dbms-guide]

## <a name="deployment-scenarios-for-sap-software-on-azure-vms"></a><a name="b3253ee3-d63b-4d74-a49b-185e76c4088e"></a>Escenarios de implementación de software de SAP en máquinas virtuales de Azure

Tiene varias opciones para implementar máquinas virtuales y sus discos asociados en Azure. Es importante comprender las diferencias entre las opciones de implementación, porque podría seguir pasos diferentes para preparar las máquinas virtuales para la implementación en función del tipo de implementación que elija.

### <a name="scenario-1-deploying-a-vm-from-the-azure-marketplace-for-sap"></a><a name="db477013-9060-4602-9ad4-b0316f8bb281"></a>Escenario 1: Implementación de una máquina virtual desde Azure Marketplace para SAP

Para implementar una máquina virtual se puede usar una imagen proporcionada por Microsoft o por un tercero en Azure Marketplace. Marketplace ofrece algunas imágenes de sistema operativo estándar de Windows Server y diferentes distribuciones de Linux. También puede implementar una imagen que incluya SKU del sistema de administración de base de datos (DBMS), por ejemplo, Microsoft SQL Server. Para más información sobre el uso de imágenes con SKU de DBMS, consulte [Implementación de DBMS de Azure Virtual Machines para SAP en NetWeaver][dbms-guide].

El diagrama de flujo siguiente muestra la secuencia de pasos específica de SAP para implementar una máquina virtual desde Azure Marketplace:

![Diagrama de flujo de la implementación de máquinas virtuales para sistemas SAP con una imagen de máquina virtual de Azure Marketplace][deployment-guide-figure-100]

#### <a name="create-a-virtual-machine-by-using-the-azure-portal"></a>Creación de una máquina virtual con Azure Portal

La manera más fácil de crear una máquina virtual nueva con una imagen de Azure Marketplace es mediante Azure Portal.

1.  Ir a <https://portal.azure.com/#create/hub>.  O bien, en el menú de Azure Portal, seleccione **+ Nuevo**.
1.  Seleccione **Proceso** y, a continuación, seleccione el tipo de sistema operativo que desea implementar. Por ejemplo, Windows Server 2012 R2, SUSE Linux Enterprise Server 12 (SLES 12), Red Hat Enterprise Linux 7.2 (RHEL 7.2) u Oracle Linux 7.2. La vista de lista predeterminada no muestra todos los sistemas operativos compatibles. Seleccione **ver todo** para obtener una lista completa. Para más información acerca de los sistemas operativos compatibles para la implementación de software de SAP, consulte la nota de SAP [1928533].
1.  En la siguiente página, revise los términos y condiciones.
1.  En la lista **Seleccionar un modelo de implementación**, seleccione **Resource Manager**.
1.  Seleccione **Crear**.

El asistente le guiará a través de los parámetros que se requieren para crear la máquina virtual junto con todos los recursos necesarios, como las interfaces de red o las cuentas de almacenamiento. Algunos de estos parámetros son:

1. **Aspectos básicos**:
   * **Name**: nombre del recurso (la máquina virtual).
   * **Tipo de disco de máquina virtual**: seleccione el tipo de disco del disco del SO. Si desea usar Premium Storage para los discos de datos, se recomienda usar Premium Storage para el disco del SO.
   * **Nombre de usuario y contraseña** o **Clave pública SSH**: escriba el nombre de usuario y la contraseña que se crearon durante el aprovisionamiento. En una máquina virtual Linux, también puede escribir la clave de Secure Shell (SSH) pública que se usa para iniciar sesión en la máquina.
   * **Suscripción**: seleccione la suscripción que quiere usar para aprovisionar la nueva máquina virtual.
   * **Grupo de recursos**: nombre del grupo de recursos de la máquina virtual. Puede escribir el nombre de un grupo de recursos nuevo o uno existente.
   * **Ubicación**: lugar en donde se implementará la nueva máquina virtual. Si desea conectar la máquina virtual con la red local, asegúrese de seleccionar la ubicación de la red virtual que conecta Azure con su red local. Para más información, consulte [Redes de Microsoft Azure][planning-guide-microsoft-azure-networking] en [Planeación e implementación de Azure Virtual Machines para SAP en NetWeaver][planning-guide].
1. **Size**:

     Lea la nota de SAP [1928533] para ver una lista de los tipos de máquinas virtuales que se admiten. Asegúrese de seleccionar el tipo de máquina virtual correcto si quiere usar Azure Premium Storage. No todos los tipos de VM son compatibles con Premium Storage. Para más información consulte [Almacenamiento: Microsoft Azure Storage y los discos de datos][planning-guide-storage-microsoft-azure-storage-and-data-disks] y [Azure Storage para cargas de trabajo de SAP](./planning-guide-storage.md) en [Planeación e implementación de Azure Virtual Machines para SAP en NetWeaver][planning-guide].

1. **Configuración**:
   * **Storage**
     * **Tipo de disco**: seleccione el tipo de disco del disco del SO. Si desea usar Premium Storage para los discos de datos, se recomienda usar Premium Storage para el disco del SO.
     * **Usar discos administrados**: si desea usar Managed Disks, seleccione Sí. Para más información sobre Managed Disks, consulte el capítulo [Managed Disks](./planning-guide-storage.md#microsoft-azure-storage-resiliency) en la guía de planeamiento.
     * **Cuenta de almacenamiento**: Seleccione una cuenta de almacenamiento existente o cree una. No todos los tipos de almacenamiento admiten la ejecución de aplicaciones SAP. Para más información sobre los tipos de almacenamiento, consulte [Estructura de almacenamiento de una máquina virtual para las implementaciones de RDBMS](./dbms_guide_general.md#65fa79d6-a85f-47ee-890b-22e794f51a64).
   * **Network**
     * **Red virtual** y **Subred**: seleccione la red virtual que está conectada a la red local si desea integrar la máquina virtual en la intranet.
     * **Dirección IP pública**: seleccione la dirección IP pública que quiere usar o escriba los parámetros para crear una dirección IP pública. Puede usar una dirección IP pública para tener acceso a la máquina virtual a través de Internet. Asegúrese de crear también un grupo de seguridad de red para ayudar a proteger el acceso a la máquina virtual.
     * **Grupo de seguridad de red**: para más información, consulte [Control del flujo de tráfico de red con grupos de seguridad de red][virtual-networks-nsg].
   * **Extensiones**: puede instalar extensiones de máquina virtual agregándolas a la implementación. No es necesario agregar extensiones en este paso. Las extensiones necesarias para la compatibilidad con SAP se instalan más adelante. Vea el capítulo [Configuración de la extensión de Azure para SAP][deployment-guide-4.5] de esta guía.
   * **Alta disponibilidad**: seleccione un conjunto de disponibilidad o escriba los parámetros para crear un conjunto de disponibilidad. Para más información, consulte [Conjuntos de disponibilidad de Azure][planning-guide-3.2.3].
   * **Supervisión**
     * **Diagnósticos de arranque**: puede seleccionar **Deshabilitar** para el diagnóstico de arranque.
     * **Diagnósticos del SO invitado**: puede seleccionar **Deshabilitar** para el diagnóstico de supervisión.

1. **Resumen**:

   Revise las opciones seleccionadas y, a continuación, seleccione **Aceptar**.

La máquina virtual se implementa en el grupo de recursos que seleccionó.

#### <a name="create-a-virtual-machine-by-using-a-template"></a>Creación de una máquina virtual con una plantilla

También puede crear una máquina virtual con una de las plantillas de SAP publicadas en el [repositorio azure-quickstart-templates de GitHub][azure-quickstart-templates-github]. O bien puede crear una máquina virtual manualmente con [Azure Portal][virtual-machines-windows-tutorial], [PowerShell][virtual-machines-ps-create-preconfigure-windows-resource-manager-vms] o la [CLI de Azure][virtual-machines-linux-tutorial].

* [**Plantilla de configuración de dos niveles (solo una máquina virtual)** (sap-2-tier-marketplace-image)][sap-templates-2-tier-marketplace-image]

  Use esta plantilla para crear un sistema de dos niveles con solo una máquina virtual.
* [**Plantilla de configuración de dos niveles (solo una máquina virtual): Managed Disks** (sap-2-tier-marketplace-image-md)][sap-templates-2-tier-marketplace-image-md]

  Use esta plantilla para crear un sistema de dos niveles con solo una máquina virtual y Managed Disks.
* [**Plantilla de configuración de tres niveles (varias máquinas virtuales)** (sap-3-tier-marketplace-image)][sap-templates-3-tier-marketplace-image]

  Use esta plantilla para crear un sistema de tres niveles con varias máquinas virtuales.
* [**Plantilla de configuración de tres niveles (varias máquinas virtuales): Managed Disks** (sap-3-tier-marketplace-image-md)][sap-templates-3-tier-marketplace-image-md]

  Use esta plantilla para crear un sistema de tres niveles con varias máquinas virtuales y Managed Disks.

En Azure Portal, especifique los siguientes parámetros para la plantilla:

1. **Aspectos básicos**:
   * **Suscripción**: suscripción que desea usar para implementar la plantilla.
   * **Grupo de recursos**: grupo de recursos que desea usar para implementar la plantilla. Puede crear un grupo de recursos o seleccionar uno en la suscripción.
   * **Ubicación**: dónde se implementará la plantilla. Si ha seleccionado un grupo de recursos, se utilizará la ubicación de ese grupo de recursos.

1. **Configuración**:
   * **Identificador de sistema SAP**: identificador del sistema SAP (SID).
   * **Tipo de SO**: sistema operativo que quiere implementar, por ejemplo, Windows Server 2012 R2, SUSE Linux Enterprise Server 12 (SLES 12), Red Hat Enterprise Linux 7.2 (RHEL 7.2) u Oracle Linux 7.2.

     La vista de lista no muestra todos los sistemas operativos compatibles. Para más información acerca de los sistemas operativos compatibles para la implementación de software de SAP, consulte la nota de SAP [1928533].
   * **Tamaño del sistema SAP**: tamaño del sistema SAP.

     La cantidad de SAPS que el sistema nuevo proporciona. Si no está seguro de cuántos SAPS necesita el sistema, consulte con el integrador de sistemas o el asociado tecnológico de SAP.
   * **Disponibilidad del sistema**: disponibilidad del sistema (solo para las plantillas de tres niveles).

     Seleccione **HA** para una configuración en la que se puede realizar una instalación de alta disponibilidad. Se crean dos servidores de base de datos y dos servidores para ABAP SAP Central Services (ASCS).
   * **Tipo de almacenamiento**: Tipo de almacenamiento que se va a usar (solo para plantillas de dos niveles).

     En los sistemas grandes, se recomienda usar Azure Premium Storage. Para más información acerca de los tipos de almacenamiento, consulte estos recursos:
      * [Use of Azure Premium SSD Storage for SAP DBMS Instance][2367194] (Uso de almacenamiento de SSD Premium de Azure para la instancia de DBMS de SAP)
      * [Estructura de almacenamiento de una máquina virtual para las implementaciones de RDBMS](./dbms_guide_general.md#65fa79d6-a85f-47ee-890b-22e794f51a64)
      * [Premium Storage: Almacenamiento de alto rendimiento para cargas de trabajo de Azure Virtual Machines][storage-premium-storage-preview-portal]
      * [Introducción a Microsoft Azure Storage][storage-introduction]
   * **Nombre de usuario de administrador** y **Contraseña de administrador**: un nombre de usuario y una contraseña.
     Se crea un nuevo usuario para iniciar sesión en la máquina virtual.
   * **New or existing subnet**: (Subred nueva o existente): determina si es necesario crear una red virtual y una subred nuevas o si se debe usar una subred existente. Si ya tiene una red virtual conectada a la red local, seleccione **Existing** (Existente).
   * **Identificador de subred**: Si quiere implementar la máquina virtual en una red virtual existente en la que tiene una subred definida a la que se debe asignar la máquina virtual, asigne un nombre al identificador de esa subred específica. El identificador suele tener este formato: /subscriptions/&lt;id. de suscripción>/resourceGroups/&lt;nombre del grupo de recursos>/providers/Microsoft.Network/virtualNetworks/&lt;nombre de red virtual>/subnets/&lt;nombre de subred>

1. **Términos y condiciones**:  
    Lea y acepte los términos legales.

1. Seleccione **Comprar**.

El agente de máquina virtual de Azure se implementa de manera predeterminada cuando se usa una imagen de Azure Marketplace.

#### <a name="configure-proxy-settings"></a>Configuración de los valores de proxy

Según cómo esté configurada la red local, quizás tenga que configurar el servidor proxy en su máquina virtual. Si la máquina virtual está conectada a la red local a través de VPN o ExpressRoute, es posible que la VM no pueda tener acceso a Internet y no pueda descargar las extensiones de VM necesarias ni recopilar información de la infraestructura de Azure para SAP Host Agent a través de la extensión de SAP para Azure. Para más información, consulte [Configuración de proxy][deployment-guide-configure-proxy].

#### <a name="join-a-domain-windows-only"></a>Unión a un dominio (solo Windows)

Si la implementación de Azure está conectada a una instancia local de Active Directory o DNS mediante una conexión de VPN de sitio a sitio de Azure o ExpressRoute (esto se denomina *entre locales* en [Planeación e implementación de Azure Virtual Machines para SAP en NetWeaver][planning-guide]), se espera que la máquina virtual se una a un dominio local. Para más información acerca de las consideraciones para esta tarea, consulte [Unión de una máquina virtual a un dominio local (solo Windows)][deployment-guide-4.3].

#### <a name="configure-vm-extension"></a><a name="ec323ac3-1de9-4c3a-b770-4ff701def65b"></a>Configuración de la extensión de máquina virtual

Para asegurarse de que SAP es compatible con el entorno, configure la extensión de Azure para SAP tal y como se describe en [Configuración de la extensión de Azure para SAP][deployment-guide-4.5]. Compruebe los requisitos previos de SAP y las versiones mínimas del kernel de SAP y de SAP Host Agent requeridas, en los recursos que se indican en [Recursos de SAP][deployment-guide-2.2].

#### <a name="vm-extension-for-sap-check"></a>Comprobación de la extensión de máquina virtual para SAP

Compruebe si la extensión de VM para SAP funciona, tal como se describe en [Comprobaciones y solución de problemas][deployment-guide-troubleshooting-chapter].

#### <a name="post-deployment-steps"></a>Pasos posteriores a la implementación

Después de crear y de implementar la máquina virtual, debe instalar los componentes de software necesarios en la máquina virtual. Debido a la secuencia de instalación de software e implementación en este tipo de implementación de máquina virtual, el software que se va a instalar ya debe estar disponible, ya sea en Azure, en otra máquina virtual o como un disco que se puede conectar. O bien, considere la posibilidad de utilizar un escenario entre locales en los que ya existe conexión con los recursos locales (recursos compartidos de instalación).

Una vez que se ha implementado la máquina virtual en Azure, siga las mismas directrices e instrucciones para instalar el software de SAP en la máquina virtual de la misma forma que lo haría en un entorno local. Para instalar el software de SAP en una máquina virtual de Azure, SAP y Microsoft recomiendan cargar y guardar los medios de instalación de SAP en los discos duros virtuales de Azure o Managed Disks, o bien crear una máquina virtual de Azure que funcione como un servidor de archivos y que contenga todos los medios de instalación de SAP necesarios.

### <a name="scenario-2-deploying-a-vm-with-a-custom-image-for-sap"></a><a name="54a1fc6d-24fd-4feb-9c57-ac588a55dff2"></a>Escenario 2: Implementación de una máquina virtual con una imagen personalizada para SAP

Como las diferentes versiones de un sistema operativo o DBMS tienen requisitos de revisión diferentes, las imágenes que encuentre en Azure Marketplace podrían no ajustarse a sus necesidades. En su lugar, quizás quiera crear una máquina virtual con su propia imagen de máquina virtual de sistema operativo o DBMS, que puede implementar de nuevo más tarde.
Los pasos para crear una imagen privada de Linux y una imagen privada de Windows son diferentes.

---
> ![Windows][Logo_Windows] Windows
>
> Para preparar una imagen de Windows que pueda usar para implementar varias máquinas virtuales, la configuración de Windows (como el nombre de host y el SID de Windows) debe estar resumida o generalizada en la máquina virtual local. Puede utilizar [sysprep](/previous-versions/windows/it-pro/windows-8.1-and-8/hh825084(v=win.10)) para ello.
>
> ![Linux][Logo_Linux] Linux
>
> Para preparar una imagen de Linux que se pueda usar para implementar varias máquinas virtuales, parte de la configuración de Linux debe estar resumida o generalizada en la máquina virtual local. Puede utilizar `waagent -deprovision` para ello. Para más información, consulte [Capturar una máquina virtual Linux en Azure][virtual-machines-linux-capture-image] y [Guía de usuario del agente de Linux de Azure][virtual-machines-linux-agent-user-guide-command-line-options].
>
>

---
Puede preparar y crear una imagen personalizada y, a continuación, usarla para crear varias máquinas virtuales nuevas. Esto se describe en [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver][planning-guide]. Puede configurar el contenido de base de datos con el administrador de aprovisionamiento de software de SAP para instalar un nuevo sistema SAP (restaura una copia de seguridad de base de datos desde un disco conectado a la máquina virtual) o puede restaurar directamente una copia de seguridad de base de datos desde Azure Storage (si DBMS lo permite). Para más información, consulte [Implementación de DBMS en Azure Virtual Machines para SAP NetWeaver][dbms-guide]. Si ya instaló un sistema SAP en la máquina virtual local (especialmente para los sistemas de dos niveles), puede adaptar la configuración del sistema SAP después de realizar la implementación de la máquina virtual de Azure mediante el procedimiento de cambio de nombre del sistema admitido por el administrador de aprovisionamiento de software de SAP (nota de SAP [1619720]). De lo contrario, puede instalar el software de SAP después de la implementación de la máquina virtual de Azure.

El diagrama de flujo siguiente muestra la secuencia de pasos específica de SAP para implementar una máquina virtual desde una imagen personalizada:

![Diagrama de flujo de la implementación de máquinas virtuales para sistemas SAP con una imagen de máquina virtual de Marketplace privado][deployment-guide-figure-300]

#### <a name="create-a-virtual-machine-by-using-the-azure-portal"></a>Creación de una máquina virtual con Azure Portal

La manera más fácil de crear una máquina virtual nueva desde una imagen de Managed Disks es mediante Azure Portal. Para obtener más información sobre cómo crear una imagen de Managed Disks, lea [Captura de una imagen administrada de una máquina virtual generalizada en Azure](../../windows/capture-image-resource.md).

1.  Ir a <https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Compute%2Fimages>. O bien, en el menú de Azure Portal, seleccione **Imágenes**.
1.  Seleccione la imagen de Managed Disks que desee implementar y haga clic en **Crear VM**.

El asistente le guiará a través de los parámetros que se requieren para crear la máquina virtual junto con todos los recursos necesarios, como las interfaces de red o las cuentas de almacenamiento. Algunos de estos parámetros son:

1. **Aspectos básicos**:
   * **Name**: nombre del recurso (la máquina virtual).
   * **Tipo de disco de máquina virtual**: seleccione el tipo de disco del disco del SO. Si desea usar Premium Storage para los discos de datos, se recomienda usar Premium Storage para el disco del SO.
   * **Nombre de usuario y contraseña** o **Clave pública SSH**: escriba el nombre de usuario y la contraseña que se crearon durante el aprovisionamiento. En una máquina virtual Linux, también puede escribir la clave de Secure Shell (SSH) pública que se usa para iniciar sesión en la máquina.
   * **Suscripción**: seleccione la suscripción que quiere usar para aprovisionar la nueva máquina virtual.
   * **Grupo de recursos**: nombre del grupo de recursos de la máquina virtual. Puede escribir el nombre de un grupo de recursos nuevo o uno existente.
   * **Ubicación**: lugar en donde se implementará la nueva máquina virtual. Si desea conectar la máquina virtual con la red local, asegúrese de seleccionar la ubicación de la red virtual que conecta Azure con su red local. Para más información, consulte [Redes de Microsoft Azure][planning-guide-microsoft-azure-networking] en [Planeación e implementación de Azure Virtual Machines para SAP en NetWeaver][planning-guide].
1. **Size**:

     Lea la nota de SAP [1928533] para ver una lista de los tipos de máquinas virtuales que se admiten. Asegúrese de seleccionar el tipo de máquina virtual correcto si quiere usar Azure Premium Storage. No todos los tipos de VM son compatibles con Premium Storage. Para más información consulte [Almacenamiento: Microsoft Azure Storage y los discos de datos][planning-guide-storage-microsoft-azure-storage-and-data-disks] y [Azure Storage para cargas de trabajo de SAP](./planning-guide-storage.md) en [Planeación e implementación de Azure Virtual Machines para SAP en NetWeaver][planning-guide].

1. **Configuración**:
   * **Storage**
     * **Tipo de disco**: seleccione el tipo de disco del disco del SO. Si desea usar Premium Storage para los discos de datos, se recomienda usar Premium Storage para el disco del SO.
     * **Usar discos administrados**: si desea usar Managed Disks, seleccione Sí. Para más información sobre Managed Disks, consulte el capítulo [Managed Disks](./planning-guide-storage.md#microsoft-azure-storage-resiliency) en la guía de planeamiento.
   * **Network**
     * **Red virtual** y **Subred**: seleccione la red virtual que está conectada a la red local si desea integrar la máquina virtual en la intranet.
     * **Dirección IP pública**: seleccione la dirección IP pública que quiere usar o escriba los parámetros para crear una dirección IP pública. Puede usar una dirección IP pública para tener acceso a la máquina virtual a través de Internet. Asegúrese de crear también un grupo de seguridad de red para ayudar a proteger el acceso a la máquina virtual.
     * **Grupo de seguridad de red**: para más información, consulte [Control del flujo de tráfico de red con grupos de seguridad de red][virtual-networks-nsg].
   * **Extensiones**: puede instalar extensiones de máquina virtual agregándolas a la implementación. No es necesario agregar extensiones en este paso. Las extensiones necesarias para la compatibilidad con SAP se instalan más adelante. Vea el capítulo [Configuración de la extensión de Azure para SAP][deployment-guide-4.5] de esta guía.
   * **Alta disponibilidad**: seleccione un conjunto de disponibilidad o escriba los parámetros para crear un conjunto de disponibilidad. Para más información, consulte [Conjuntos de disponibilidad de Azure][planning-guide-3.2.3].
   * **Supervisión**
     * **Diagnósticos de arranque**: puede seleccionar **Deshabilitar** para el diagnóstico de arranque.
     * **Diagnósticos del SO invitado**: puede seleccionar **Deshabilitar** para el diagnóstico de supervisión.

1. **Resumen**:

   Revise las opciones seleccionadas y, a continuación, seleccione **Aceptar**.

La máquina virtual se implementa en el grupo de recursos que seleccionó.

#### <a name="create-a-virtual-machine-by-using-a-template"></a>Creación de una máquina virtual con una plantilla

Para crear una implementación con una imagen de sistema operativo privada mediante Azure Portal, use una de las siguientes plantillas de SAP. Estas plantillas están publicadas en el [repositorio azure-quickstart-templates de GitHub][azure-quickstart-templates-github]. También puede crear una máquina virtual manualmente con [PowerShell][virtual-machines-upload-image-windows-resource-manager].

* [**Plantilla de configuración de dos niveles (solo una máquina virtual)** (sap-2-tier-user-image)][sap-templates-2-tier-user-image]

  Use esta plantilla para crear un sistema de dos niveles con solo una máquina virtual.
* [**Plantilla de configuración de dos niveles (solo una máquina virtual): imagen de Managed Disks** (sap-2-tier-user-image-md)][sap-templates-2-tier-user-image-md]

  Use esta plantilla para crear un sistema de dos niveles con solo una máquina virtual y una imagen de Managed Disks.
* [**Plantilla de configuración de tres niveles (varias máquinas virtuales)** (sap-3-tier-user-image)][sap-templates-3-tier-user-image]

  Use esta plantilla para crear un sistema de tres niveles con varias máquinas virtuales o su propia imagen de sistema operativo.
* [**Plantilla de configuración de tres niveles (varias máquinas virtuales): imagen de Managed Disks** (sap-3-tier-user-image-md)][sap-templates-3-tier-user-image-md]

  Use esta plantilla para crear un sistema de tres niveles con varias máquinas virtuales o su propia imagen de SO y una imagen de Managed Disks.

En Azure Portal, especifique los siguientes parámetros para la plantilla:

1. **Aspectos básicos**:
   * **Suscripción**: suscripción que desea usar para implementar la plantilla.
   * **Grupo de recursos**: grupo de recursos que desea usar para implementar la plantilla. Puede crear un grupo de recursos o seleccionar uno en la suscripción.
   * **Ubicación**: dónde se implementará la plantilla. Si ha seleccionado un grupo de recursos, se utilizará la ubicación de ese grupo de recursos.
1. **Configuración**:
   * **Identificador de sistema SAP**: identificador del sistema SAP.
   * **Tipo de SO**: sistema operativo que quiere implementar (Windows o Linux).
   * **Tamaño del sistema SAP**: tamaño del sistema SAP.

     La cantidad de SAPS que el sistema nuevo proporciona. Si no está seguro de cuántos SAPS necesita el sistema, consulte con el integrador de sistemas o el asociado tecnológico de SAP.
   * **Disponibilidad del sistema**: disponibilidad del sistema (solo para las plantillas de tres niveles).

     Seleccione **HA** para una configuración en la que se puede realizar una instalación de alta disponibilidad. Se crean dos servidores de base de datos y dos servidores para ASCS.
   * **Tipo de almacenamiento**: Tipo de almacenamiento que se va a usar (solo para plantillas de dos niveles).

     En los sistemas grandes, se recomienda usar Azure Premium Storage. Para más información acerca de los tipos de almacenamiento, consulte estos recursos:
      * [Use of Azure Premium SSD Storage for SAP DBMS Instance][2367194] (Uso de almacenamiento de SSD Premium de Azure para la instancia de DBMS de SAP)
      * [Estructura de almacenamiento de una máquina virtual para las implementaciones de RDBMS](./dbms_guide_general.md#65fa79d6-a85f-47ee-890b-22e794f51a64)
      * [Premium Storage: Almacenamiento de alto rendimiento para cargas de trabajo de la máquina virtual de Azure][storage-premium-storage-preview-portal]
      * [Introducción a Microsoft Azure Storage][storage-introduction]
   * **User Image VHD URI** (URI del VHD de la imagen del usuario) (solo plantilla de imagen de disco no administrado): identificador URI del disco duro virtual de la imagen de SO privada; por ejemplo, https://&lt;nombreDeCuenta&gt;.blob.core.windows.net/vhds/userimage.vhd.
   * **User image storage account** (Cuenta de almacenamiento de imagen de usuario) (solo plantilla de imagen de disco no administrado): nombre de la cuenta de almacenamiento donde se almacena la imagen de sistema operativo privada, por ejemplo, &lt;nombreDeCuenta&gt; en https://&lt;nombreDeCuenta&gt;.blob.core.windows.net/vhds/userimage.vhd.
   * **userImageId** (solo plantilla de imagen de disco administrado): Identificador de la imagen de disco administrado que quiere usar
   * **Nombre de usuario de administrador** y **Contraseña de administrador**: nombre de usuario y contraseña.

     Se crea un nuevo usuario para iniciar sesión en la máquina virtual.
   * **New or existing subnet**: (Subred nueva o existente): determina si es necesario crear una red virtual y una subred nuevas o si se debe usar una subred existente. Si ya tiene una red virtual conectada a la red local, seleccione **Existing** (Existente).
   * **Identificador de subred**: Si quiere implementar la máquina virtual en una red virtual existente en la que tiene una subred definida a la que se debe asignar la máquina virtual, asigne un nombre al identificador de esa subred específica. El identificador suele tener este formato: /subscriptions/&lt;id. de suscripción>/resourceGroups/&lt;nombre del grupo de recursos>/providers/Microsoft.Network/virtualNetworks/&lt;nombre de red virtual>/subnets/&lt;nombre de subred>

1. **Términos y condiciones**:  
    Lea y acepte los términos legales.

1. Seleccione **Comprar**.

#### <a name="install-the-vm-agent-linux-only"></a>Instalación del agente de máquina virtual (solo Linux)

Para utilizar las plantillas que se describen en la sección anterior, el agente de Linux ya debe estar instalado en la imagen de usuario, o se producirá un error en la implementación. Descargue e instale el agente de máquina virtual en la imagen de usuario, tal y como se describe en [Descarga, instalación y habilitación del agente de máquina virtual de Azure][deployment-guide-4.4]. Si no usa las plantillas, también puede instalar más adelante el agente de máquina virtual.

#### <a name="join-a-domain-windows-only"></a>Unión a un dominio (solo Windows)

Si la implementación de Azure está conectada a una instancia local de Active Directory o DNS mediante una conexión de VPN de sitio a sitio de Azure o Azure ExpressRoute (esto se denomina *entre entornos locales* en [Planeación e implementación de Azure Virtual Machines para SAP en NetWeaver][planning-guide]), se espera que la máquina virtual se una a un dominio local. Para más información acerca de las consideraciones para este paso, consulte [Unión de una máquina virtual a un dominio local (solo Windows)][deployment-guide-4.3].

#### <a name="configure-proxy-settings"></a>Configuración de los valores de proxy

Según cómo esté configurada la red local, quizás tenga que configurar el servidor proxy en su máquina virtual. Si la máquina virtual está conectada a la red local a través de VPN o ExpressRoute, es posible que la VM no pueda tener acceso a Internet y no pueda descargar las extensiones de VM necesarias ni recopilar información de la infraestructura de Azure para SAP Host Agent a través de la extensión de SAP para Azure, consulte [Configuración de proxy][deployment-guide-configure-proxy].

#### <a name="configure-azure-vm-extension-for-sap"></a>Configuración de la extensión de máquina virtual de Azure para SAP

Para asegurarse de que SAP es compatible con el entorno, configure la extensión de Azure para SAP tal y como se describe en [Configuración de la extensión de Azure para SAP][deployment-guide-4.5]. Compruebe los requisitos previos de SAP y las versiones mínimas del kernel de SAP y de SAP Host Agent requeridas, en los recursos que se indican en [Recursos de SAP][deployment-guide-2.2].

#### <a name="sap-vm-extension-check"></a>Comprobación de la extensión de máquina virtual para SAP

Compruebe si la extensión de VM para SAP funciona, tal como se describe en [Comprobaciones y solución de problemas][deployment-guide-troubleshooting-chapter].


### <a name="scenario-3-moving-an-on-premises-vm-by-using-a-non-generalized-azure-vhd-with-sap"></a><a name="a9a60133-a763-4de8-8986-ac0fa33aa8c1"></a>Escenario 3: Traslado de una máquina virtual local con un VHD no generalizado de Azure con SAP

En este escenario, tiene previsto mover un sistema SAP específico desde un entorno local a Azure. Esto puede hacerse cargando a Azure el disco duro virtual que contiene el sistema operativo, los archivos binarios de SAP y los archivos binarios de DBMS, así como los discos duros virtuales con los archivos de registro y de datos del DBMS. A diferencia del escenario descrito en [Escenario 2: Implementación de una máquina virtual con una imagen personalizada para SAP][deployment-guide-3.3], en este caso se mantiene el nombre de host, el SID de SAP y las cuentas de usuario de SAP en la máquina virtual de Azure, dado que están configurados en el entorno local. No es necesario generalizar el sistema operativo. Este escenario se da con mayor frecuencia entre entornos locales, donde una parte del panorama SAP se ejecuta en un entorno local y otra parte del mismo se ejecuta en Azure.

En este escenario, el agente de máquina virtual **no** se instala automáticamente durante la implementación. Como el agente de máquina virtual y la extensión de Azure para SAP son un requisito previo para ejecutar SAP en NetWeaver en Azure, debe descargar, instalar y habilitar los dos componentes manualmente después de crear la máquina virtual.

Para más información acerca del agente de máquina virtual de Azure, consulte los siguientes recursos.

---
> ![Windows][Logo_Windows] Windows
>
> [Información general del agente de máquina virtual de Azure][virtual-machines-windows-agent-user-guide]
>
> ![Linux][Logo_Linux] Linux
>
> [Guía de usuario del Agente de Linux de Azure][virtual-machines-linux-agent-user-guide]
>
>

---

El siguiente diagrama de flujo muestra la secuencia de pasos para trasladar una máquina virtual local mediante un disco duro virtual de Azure no generalizado:

![Diagrama de flujo de la implementación de máquinas virtuales para sistemas SAP mediante un disco de máquina virtual][deployment-guide-figure-400]

Si el disco ya está cargado y definido en Azure (consulte [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver][planning-guide]), realice las tareas que se describen en las próximas secciones.

#### <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Para crear una implementación con un disco de sistema operativo privado mediante Azure Portal, use la plantilla de SAP que está publicada en el [repositorio azure-quickstart-templates de GitHub][azure-quickstart-templates-github]. También puede crear una máquina virtual manualmente con PowerShell.

* [**Plantilla de configuración de dos niveles (solo una máquina virtual)** (sap-2-tier-user-disk)][sap-templates-2-tier-os-disk]

  Use esta plantilla para crear un sistema de dos niveles con solo una máquina virtual.
* [**Plantilla de configuración de dos niveles (solo una máquina virtual): Managed Disks** (sap-2-tier-user-disk-md)][sap-templates-2-tier-os-disk-md]

  Use esta plantilla para crear un sistema de dos niveles con solo una máquina virtual y Managed Disks.

En Azure Portal, especifique los siguientes parámetros para la plantilla:

1. **Aspectos básicos**:
   * **Suscripción**: suscripción que desea usar para implementar la plantilla.
   * **Grupo de recursos**: grupo de recursos que desea usar para implementar la plantilla. Puede crear un grupo de recursos o seleccionar uno en la suscripción.
   * **Ubicación**: dónde se implementará la plantilla. Si ha seleccionado un grupo de recursos, se utilizará la ubicación de ese grupo de recursos.
1. **Configuración**:
   * **Identificador de sistema SAP**: identificador del sistema SAP.
   * **Tipo de SO**: sistema operativo que quiere implementar (Windows o Linux).
   * **Tamaño del sistema SAP**: tamaño del sistema SAP.

     La cantidad de SAPS que el sistema nuevo proporciona. Si no está seguro de cuántos SAPS necesita el sistema, consulte con el integrador de sistemas o el asociado tecnológico de SAP.
   * **Tipo de almacenamiento**: Tipo de almacenamiento que se va a usar (solo para plantillas de dos niveles).

     En los sistemas grandes, se recomienda usar Azure Premium Storage. Para más información acerca de los tipos de almacenamiento, consulte estos recursos:
      * [Use of Azure Premium SSD Storage for SAP DBMS Instance][2367194] (Uso de almacenamiento de SSD Premium de Azure para la instancia de DBMS de SAP)
      * [Estructura de almacenamiento de una máquina virtual para las implementaciones de RDBMS](./dbms_guide_general.md#65fa79d6-a85f-47ee-890b-22e794f51a64)
      * [Premium Storage: Almacenamiento de alto rendimiento para cargas de trabajo de Azure Virtual Machines][storage-premium-storage-preview-portal]
      * [Introducción a Microsoft Azure Storage][storage-introduction]
   * **OS disk VHD URI** (URI del VHD de disco de SO) (solo plantilla de disco no administrado): identificador URI del disco de sistema operativo privado; por ejemplo, https://&lt;nombreDeCuenta&gt;.blob.core.windows.net/vhds/osdisk.vhd.
   * **Identificador de Managed Disks del SO** (solo plantilla de disco administrado): Identificador del disco del SO de Managed Disks, /subscriptions/92d102f7-81a5-4df7-9877-54987ba97dd9/resourceGroups/group/providers/Microsoft.Compute/disks/WIN.
   * **New or existing subnet**: (Subred nueva o existente): determina si es necesario crear una red virtual y una subred nuevas o si se debe usar una subred existente. Si ya tiene una red virtual conectada a la red local, seleccione **Existing** (Existente).
   * **Identificador de subred**: Si quiere implementar la máquina virtual en una red virtual existente en la que tiene una subred definida a la que se debe asignar la máquina virtual, asigne un nombre al identificador de esa subred específica. El identificador suele tener este formato: /subscriptions/&lt;id. de suscripción>/resourceGroups/&lt;nombre del grupo de recursos>/providers/Microsoft.Network/virtualNetworks/&lt;nombre de red virtual>/subnets/&lt;nombre de subred>

1. **Términos y condiciones**:  
    Lea y acepte los términos legales.

1. Seleccione **Comprar**.

#### <a name="install-the-vm-agent"></a>Instalar el agente de VM

Para usar las plantillas que se describen en la sección anterior, el agente de máquina virtual debe estar instalado en el disco de sistema operativo, o se producirá un error en la implementación. Descargue e instale el agente de máquina virtual en la máquina virtual, tal y como se describe en [Descarga, instalación y habilitación del agente de máquina virtual de Azure][deployment-guide-4.4].

Si no usa las plantillas descritas en la sección anterior, también puede instalar más adelante el agente de máquina virtual.

#### <a name="join-a-domain-windows-only"></a>Unión a un dominio (solo Windows)

Si la implementación de Azure está conectada a una instancia local de Active Directory o DNS mediante una conexión de VPN de sitio a sitio de Azure o ExpressRoute (esto se denomina *entre locales* en [Planeación e implementación de Azure Virtual Machines para SAP en NetWeaver][planning-guide]), se espera que la máquina virtual se una a un dominio local. Para más información acerca de las consideraciones para esta tarea, consulte [Unión de una máquina virtual a un dominio local (solo Windows)][deployment-guide-4.3].

#### <a name="configure-proxy-settings"></a>Configuración de los valores de proxy

Según cómo esté configurada la red local, quizás tenga que configurar el servidor proxy en su máquina virtual. Si la máquina virtual está conectada a la red local a través de VPN o ExpressRoute, es posible que la VM no pueda tener acceso a Internet y no pueda descargar las extensiones de VM necesarias ni recopilar información de la infraestructura de Azure para SAP Host Agent a través de la extensión de SAP para Azure, consulte [Configuración de proxy][deployment-guide-configure-proxy].

#### <a name="configure-azure-vm-extension-for-sap"></a>Configuración de la extensión de máquina virtual de Azure para SAP

Para asegurarse de que SAP es compatible con el entorno, configure la extensión de Azure para SAP tal y como se describe en [Configuración de la extensión de Azure para SAP][deployment-guide-4.5]. Compruebe los requisitos previos de SAP y las versiones mínimas del kernel de SAP y de SAP Host Agent requeridas, en los recursos que se indican en [Recursos de SAP][deployment-guide-2.2].

#### <a name="sap-vm-check"></a>Comprobación de la máquina virtual para SAP

Compruebe si la extensión de VM para SAP funciona, tal como se describe en [Comprobaciones y solución de problemas][deployment-guide-troubleshooting-chapter].

## <a name="update-the-configuration-of-azure-extension-for-sap"></a>Actualización de la configuración de la extensión de Azure para SAP

Actualice la configuración de la extensión de Azure para SAP en cualquiera de estos escenarios:
* El equipo conjunto de Microsoft y SAP extiende las funcionalidades de la extensión de máquina virtual y solicita más o menos contenedores.
* Microsoft presenta una versión nueva de la infraestructura de Azure subyacente que brinda los datos y la extensión de Azure para SAP debe adaptarse a esos cambios.
* Puede agregar más discos de datos adicionales a la máquina virtual de Azure, o bien quitar uno. En este caso, actualice la colección de datos relacionados con el almacenamiento. Agregar o eliminar puntos de conexión o asignar direcciones IP a una máquina virtual no afecta a la configuración de la extensión.
* Cambia el tamaño de la máquina virtual de Azure, por ejemplo, de A5 a cualquier otro tamaño de máquina virtual.
* Agrega interfaces de red nuevas a la máquina virtual de Azure.

Para actualizar los ajustes, actualice la configuración de la extensión de Azure para SAP siguiendo los pasos descritos en [Configuración de la extensión de Azure para SAP][deployment-guide-4.5].

## <a name="detailed-tasks-for-sap-software-deployment"></a>Tareas detalladas para la implementación de software de SAP

En esta sección se detallan los pasos para realizar tareas específicas en el proceso de configuración e implementación.

### <a name="deploy-azure-powershell-cmdlets"></a><a name="604bcec2-8b6e-48d2-a944-61b0f5dee2f7"></a>Implementación de cmdlets de Azure PowerShell

Siga los pasos descritos en el artículo [Instalar el módulo de Azure PowerShell](/powershell/azure/install-az-ps)

Compruebe periódicamente si existen actualizaciones para los cmdlets de PowerShell, que normalmente se actualizan mensualmente. Siga los pasos descritos en [este](/powershell/azure/install-az-ps#update-the-azure-powershell-module) artículo. Salvo que se indique lo contrario en las notas de SAP [1928533] o [2015553], le recomendamos que trabaje con la versión más reciente de los cmdlets de Azure PowerShell.

Para comprobar la versión de los cmdlets de Azure PowerShell que están instalados en su equipo, ejecute este comando de PowerShell:

```powershell
(Get-Module Az.Compute).Version
```

### <a name="deploy-azure-cli"></a><a name="1ded9453-1330-442a-86ea-e0fd8ae8cab3"></a>Implementación de la CLI de Azure

Siga los pasos descritos en el artículo [Instalación de la CLI de Azure](/cli/azure/install-azure-cli)

Compruebe regularmente si hay actualizaciones de la CLI de Azure, que normalmente se actualiza mensualmente.

Para comprobar la versión de la CLI de Azure que está instalada en el equipo, ejecute este comando:

```console
az --version
```

### <a name="join-a-vm-to-an-on-premises-domain-windows-only"></a><a name="31d9ecd6-b136-4c73-b61e-da4a29bbc9cc"></a>Unión de una máquina virtual a un dominio local (solo Windows)

Si implementa máquinas virtuales de SAP en un escenario entre entornos locales en los que las instancias de Active Directory y DNS locales se extienden a Azure, se espera que las máquinas virtuales estén unidas a un dominio local. Los pasos detallados para unir una máquina virtual a un dominio local y el software adicional que se requiere para ser miembro de un dominio local varía según el cliente. Por lo general, para unir una máquina virtual a un dominio local, debe instalar software adicional, como software antimalware, de copia de seguridad o de supervisión.

En este escenario, también tiene que asegurarse de que si se fuerza la configuración de proxy de Internet cuando una máquina virtual se une a un dominio en su entorno, la cuenta de sistema local de Windows (S-1-5-18) en la máquina virtual invitada tiene la misma configuración de proxy. La manera más sencilla es obligar al servidor proxy mediante una directiva de grupo de dominio que se aplica a los sistemas de del dominio.

### <a name="download-install-and-enable-the-azure-vm-agent"></a><a name="c7cbb0dc-52a4-49db-8e03-83e7edc2927d"></a>Descarga, instalación y habilitación del agente de máquina virtual de Azure

Para las máquinas virtuales que se implementan desde una imagen de sistema operativo que no está generalizada (por ejemplo, una imagen que no se origina en la herramienta de preparación del sistema de Windows o sysprep), debe descargar, instalar y habilitar manualmente el agente de máquina virtual de Azure.

Si implementa una máquina virtual desde Azure Marketplace, este paso no es necesario. Las imágenes de Azure Marketplace ya tienen el agente de máquina virtual de Azure.

#### <a name="windows"></a><a name="b2db5c9a-a076-42c6-9835-16945868e866"></a>Windows

1. Descargue el agente de máquina virtual de Azure:
   1.  Descargue el [paquete del instalador del agente de máquina virtual de Azure](https://go.microsoft.com/fwlink/?LinkId=394789).
   1.  Almacene el paquete MSI del agente de máquina virtual localmente en su equipo o en un servidor.
1. Instale el agente de máquina virtual de Azure:
   1.  Conéctese a la máquina virtual de Azure mediante el Protocolo de Escritorio remoto (RDP).
   1.  Abra una ventana del Explorador de Windows en la máquina virtual y seleccione el directorio de destino del archivo MSI del agente de máquina virtual.
   1.  Arrastre el archivo MSI del instalador del agente de máquina virtual de Azure desde su equipo o servidor local y suéltelo en el directorio de destino en la máquina virtual.
   1.  Haga doble clic en el archivo MSI en la máquina virtual.
1. En el caso de las máquinas virtuales unidas a dominios locales, asegúrese de que las posibles configuraciones de proxy de Internet se apliquen también a la cuenta de sistema local de Windows (S-1-5-18) en la máquina virtual, tal y como se describe en [Configuración de proxy][deployment-guide-configure-proxy]. El agente de máquina virtual se ejecuta en este contexto y necesita poder conectarse a Azure.

No se necesita ninguna interacción del usuario para actualizar el agente de máquina virtual de Azure. El agente de máquina virtual se actualiza automáticamente y no requiere que la máquina virtual se reinicie.

#### <a name="linux"></a><a name="6889ff12-eaaf-4f3c-97e1-7c9edc7f7542"></a>Linux

Use los comandos siguientes para instalar el agente de máquina virtual para Linux:

* **SUSE Linux Enterprise Server (SLES)**

  ```console
  sudo zypper install WALinuxAgent
  ```

* **Red Hat Enterprise Linux (RHEL) u Oracle Linux**

  ```console
  sudo yum install WALinuxAgent
  ```

Si el agente ya está instalado, para actualizar el agente de Linux de Azure, siga los pasos que se indican en [Actualización del agente Linux de Azure en una máquina virtual a la última versión desde GitHub][virtual-machines-linux-update-agent].

### <a name="configure-the-proxy"></a><a name="baccae00-6f79-4307-ade4-40292ce4e02d"></a>Configuración de proxy

Los pasos que se siguen para configurar el proxy en Windows son diferentes en Linux.

#### <a name="windows"></a>Windows

La configuración de proxy debe establecerse correctamente para que la cuenta de sistema local tenga acceso a Internet. Si la configuración de proxy no se establece mediante directiva de grupo, puede configurar las opciones de la cuenta de sistema local.

1. Vaya a **Inicio**, escriba **gpedit.msc** y seleccione **ENTRAR**.
1. Seleccione **Configuración del equipo** > **Plantillas administrativas** > **Componentes de Windows** > **Internet Explorer**. Asegúrese de que la opción **Configuración de proxy por equipo y no por usuario** esté desactivada o no configurada.
1. En el **Panel de Control**, vaya a **Centro de redes y recursos compartidos** > **Opciones de Internet**.
1. En la pestaña **Conexiones**, seleccione el botón **Configuración de LAN**.
1. Desactive la casilla **Detectar la configuración automáticamente**.
1. Active la casilla **Usar un servidor proxy para la LAN** y escriba el puerto y la dirección del servidor proxy.
1. Seleccione el botón **Advanced** (Opciones avanzadas).
1. En el cuadro **Excepciones**, escriba la dirección IP **168.63.129.16**. Seleccione **Aceptar**.

#### <a name="linux"></a>Linux

Configure el proxy correcto en el archivo de configuración del agente invitado de Microsoft Azure, que está en \\etc\\waagent.conf.

Establezca los siguientes parámetros:

1. **HTTP proxy host** (Host de proxy HTTP). Por ejemplo, establézcalo en **proxy.corp.local**.

   ```console
   HttpProxy.Host=<proxy host>

   ```

1. **HTTP proxy port** (Puerto de proxy HTTP). Por ejemplo, establézcalo en **80**.

   ```console
   HttpProxy.Port=<port of the proxy host>

   ```

1. Reinicie el agente.

   ```console
   sudo service waagent restart
   ```

La configuración de proxy en \\etc\\waagent.conf también se aplica a las extensiones de máquina virtual requeridas. Si desea usar los repositorios de Azure, asegúrese de que el tráfico a estos repositorios no pase por la intranet local. Si creó rutas definidas por el usuario para habilitar la tunelización forzada, asegúrese de agregar una ruta que enrute el tráfico hacia los repositorios directamente a Internet y no a través de su conexión de sitio a sitio.

* **SLES**

  También es necesario agregar rutas para las direcciones IP que aparecen en \\etc\\regionserverclnt.cfg. La ilustración siguiente muestra un ejemplo:

  ![Tunelización forzada][deployment-guide-figure-50]


* **RHEL**

  También es necesario que agregue rutas para las direcciones IP de los hosts que aparecen en \\etc\\yum.repos.d\\rhui-load-balancers. La ilustración anterior muestra un ejemplo.

* **Oracle Linux**

  No hay ningún repositorio para Oracle Linux en Azure. Debe configurar sus propios repositorios para Oracle Linux o usar los repositorios públicos.

Para más información sobre las rutas definidas por el usuario, consulte [Rutas definidas por el usuario y reenvío IP][virtual-networks-udr-overview].

### <a name="configure-the-azure-extension-for-sap"></a><a name="d98edcd3-f2a1-49f7-b26a-07448ceb60ca"></a>Configuración de la extensión de Azure para SAP

> [!NOTE]
> Declaración general de soporte técnico: Abra siempre un incidente con SAP en el componente BC-OP-NT-AZR para Windows o BC-OP-LNX-AZR si necesita soporte técnico para la extensión de Azure para SAP.
> Hay ingenieros de Soporte técnico de Microsoft dedicados que trabajan en el sistema de soporte de SAP para ayudar a nuestros clientes conjuntos.

Cuando la máquina virtual esté preparada tal y como se describe en [Escenarios de implementación de máquinas virtuales para SAP en Azure][deployment-guide-3], el Agente de máquina virtual de Azure se instala en la máquina virtual. El siguiente paso es implementar la extensión de Azure para SAP, que está disponible en el repositorio de extensiones de Azure, en los centros de datos globales de Azure. Para más información, consulte [Planeación e implementación de Azure Virtual Machines para SAP en NetWeaver][planning-guide-9.1].

Estamos en proceso de lanzar una nueva versión de la extensión de Azure para SAP. La nueva extensión usa la identidad asignada por el sistema de la máquina virtual para obtener información acerca de los discos conectados, las interfaces de red y la propia máquina virtual. Para poder tener acceso a estos recursos, la identidad del sistema de la máquina virtual necesita permiso de lectura para la máquina virtual, el disco del sistema operativo, los discos de datos y las interfaces de red. Actualmente, se recomienda instalar la nueva extensión en los escenarios siguientes:

1. Quiere instalar la extensión con Terraform, plantillas de Azure Resource Manager o con otros medios distintos de la CLI de Azure o Azure PowerShell.
1. Quiere instalar la extensión en SUSE SLES 15 o superior.
1. El servicio de soporte técnico de Microsoft o SAP le pide que instale la nueva extensión.
1. Quiere usar un disco Ultra de Azure o Managed Disks estándar

Para estos escenarios, siga los pasos descritos en el capítulo [Configuración de la nueva extensión de Azure para SAP con Azure PowerShell][deployment-guide-configure-new-extension-ps] para Azure PowerShell o [Configuración de la nueva extensión de Azure para SAP con la CLI de Azure][deployment-guide-configure-new-extension-cli] para la CLI de Azure.

Siga [Azure PowerShell][deployment-guide-4.5.1] o [CLI de Azure][deployment-guide-4.5.2] para instalar y configurar la versión estándar de la extensión de Azure para SAP.

#### <a name="azure-powershell-for-linux-and-windows-vms"></a><a name="987cf279-d713-4b4c-8143-6b11589bb9d4"></a>Azure PowerShell para máquinas virtuales Linux y con Windows

Para instalar la extensión de Azure para SAP mediante PowerShell:

1. Asegúrese de tener instalada la versión más reciente del cmdlet de Azure PowerShell. Para más información, consulte [Implementación de cmdlets de Azure PowerShell][deployment-guide-4.1].  
1. Ejecute el siguiente cmdlet de PowerShell.
    Para ver la lista de entornos disponibles, ejecute el cmdlet `Get-AzEnvironment`. Si desea usar una instancia global de Azure, el entorno es **AzureCloud**. Para Azure China 21Vianet, seleccione **AzureChinaCloud**.

    ```powershell
    $env = Get-AzEnvironment -Name <name of the environment>
    Connect-AzAccount -Environment $env
    Set-AzContext -SubscriptionName <subscription name>

    Set-AzVMAEMExtension -ResourceGroupName <resource group name> -VMName <virtual machine name>
    ```

Después de proporcionar los datos de la cuenta, el script implementa las extensiones requeridas y habilita las funciones necesarias. Esto puede tardar varios minutos.
Para más información sobre `Set-AzVMAEMExtension`, consulte [Set-AzVMAEMExtension][msdn-set-Azvmaemextension].

![Ejecución correcta del cmdlet de Azure específico de SAP Set-AzVMAEMExtension][deployment-guide-figure-900]

La configuración `Set-AzVMAEMExtension` realiza todos los pasos para configurar la recopilación de datos de host para SAP.

La salida del script incluye la información siguiente:

* Confirmación de que se ha configurado la recopilación de datos para el disco del SO y todos los discos de datos adicionales.
* Los próximos dos mensajes confirman la configuración de las métricas de almacenamiento de una cuenta de almacenamiento específica.
* Una línea de la salida muestra el estado de la actualización real de la configuración de la extensión de máquina virtual para SAP.
* Otra línea de la salida confirma que la configuración se implementó o se actualizó.
* La última línea de la salida es informativa. Muestra las opciones para probar la configuración de la extensión de máquina virtual para SAP.
* Para comprobar que todos los pasos de la configuración de la extensión de máquina virtual de Azure para SAP se ejecutaron correctamente y que la infraestructura de Azure proporciona los datos necesarios, compruebe si la extensión de Azure para SAP está preparada, tal como se describe en [Comprobación de la preparación de la extensión de Azure para SAP][deployment-guide-5.1].
* Espere de 15 a 30 minutos para que Diagnósticos de Azure recopile los datos pertinentes.

#### <a name="azure-cli-for-linux-vms"></a><a name="408f3779-f422-4413-82f8-c57a23b4fc2f"></a>CLI de Azure para máquinas virtuales de Linux

Para instalar la extensión de Azure para SAP mediante la CLI de Azure:

   1. Instale la CLI de Azure clásica tal y como se describe en [Instalación de la CLI clásica de Azure][azure-cli].
   1. Inicie sesión con su cuenta de Azure:

      ```console
      azure login
      ```

   1. Cambie al modo de Azure Resource Manager:

      ```console
      azure config mode arm
      ```

   1. Habilite la extensión de Azure para SAP:

      ```console
      azure vm enable-aem <resource-group-name> <vm-name>
      ```

1. Instalación mediante la CLI de Azure 2.0

   1. Instale la CLI de Azure 2.0 tal y como se describe en [Instalación de la CLI de Azure 2.0][azure-cli-2].
   1. Inicie sesión con su cuenta de Azure:

      ```azurecli
      az login
      ```

   1. Instalación de la extensión AEM de la CLI de Azure
  
      ```azurecli
      az extension add --name aem
      ```
  
   1. Instalación de la extensión con
  
      ```azurecli
      az vm aem set -g <resource-group-name> -n <vm name>
      ```

1. Compruebe que la extensión de Azure para SAP esté activa en la máquina virtual Linux de Azure. Compruebe si el archivo \\var\\lib\\AzureEnhancedMonitor\\PerfCounters existe. Si existe, en un símbolo del sistema, ejecute este comando para mostrar la información recopilada por la extensión de Azure para SAP:

   ```console
   cat /var/lib/AzureEnhancedMonitor/PerfCounters
   ```

   El resultado tendrá un aspecto similar al siguiente:

   ```output
   ...
   2;cpu;Current Hw Frequency;;0;2194.659;MHz;60;1444036656;saplnxmon;
   2;cpu;Max Hw Frequency;;0;2194.659;MHz;0;1444036656;saplnxmon;
   ...
   ```

#### <a name="configure-the-new-azure-extension-for-sap-with-azure-powershell"></a><a name="2ad55a0d-9937-4943-9dd2-69bc2b5d3de0"></a>Configuración de la nueva extensión de Azure para SAP con Azure PowerShell

La nueva extensión de VM para SAP usa una identidad administrada asignada a la VM para acceder a los datos de supervisión y configuración de la VM. Para instalar la nueva extensión de Azure para SAP mediante PowerShell, primero debe asignar dicha identidad a la VM y conceder a dicha identidad acceso a todos los recursos que usa esa VM; por ejemplo, discos e interfaces de red.

> [!NOTE]
> Los pasos siguientes requieren privilegios de propietario para el grupo de recursos o en recursos individuales (máquinas virtuales, discos de datos, etc.).

1. Asegúrese de usar el agente del host 7.21 PL 47 o posterior.
1. Asegúrese de desinstalar la versión actual de la extensión de VM para SAP. No se admite la instalación de ambas versiones de la extensión de VM para SAP en la misma máquina virtual.
1. Asegúrese de tener instalada la versión más reciente del cmdlet de Azure PowerShell (al menos 4.3.0). Para más información, consulte [Implementación de cmdlets de Azure PowerShell][deployment-guide-4.1].
1. Ejecute el siguiente cmdlet de PowerShell.
    Para ver la lista de entornos disponibles, ejecute el cmdlet `Get-AzEnvironment`. Si desea usar una instancia global de Azure, el entorno es **AzureCloud**. Para Azure China 21Vianet, seleccione **AzureChinaCloud**.

    ```powershell
    $env = Get-AzEnvironment -Name <name of the environment>
    Connect-AzAccount -Environment $env
    Set-AzContext -SubscriptionName <subscription name>

    Set-AzVMAEMExtension -ResourceGroupName <resource group name> -VMName <virtual machine name> -InstallNewExtension
    ```

#### <a name="configure-the-new-azure-extension-for-sap-with-azure-cli"></a><a name="c8749c24-fada-42ad-b114-f9aae2dc37da"></a>Configuración de la nueva extensión de Azure para SAP con la CLI de Azure

La nueva extensión de VM para SAP usa una identidad administrada asignada a la VM para acceder a los datos de supervisión y configuración de la VM. Para instalar la nueva extensión de Azure para SAP mediante la CLI de Azure, primero debe asignar dicha identidad a la VM y conceder a dicha identidad acceso a todos los recursos que usa esa VM; por ejemplo, discos e interfaces de red.

> [!NOTE]
> Los pasos siguientes requieren privilegios de propietario para el grupo de recursos o en recursos individuales (máquinas virtuales, discos de datos, etc.).

1. Asegúrese de usar el agente del host 7.21 PL 47 o posterior.
1. Asegúrese de desinstalar la versión actual de la extensión de VM para SAP. No se admite la instalación de ambas versiones de la extensión de VM para SAP en la misma máquina virtual.
1. Instale la CLI de Azure 2.0 tal y como se describe en [Instalación de la CLI de Azure 2.0][azure-cli-2].

1. Inicie sesión con su cuenta de Azure:

   ```azurecli
   az login
   ```

1. Siga los pasos del artículo [Configurar identidades administradas para recursos de Azure en una VM de Azure con la CLI de Azure][qs-configure-cli-windows-vm] para habilitar una identidad administrada asignada por el sistema en la VM. Las identidades administradas asignadas por el usuario no son compatibles con la extensión de VM para SAP. Sin embargo, puede habilitar tanto una identidad asignada por el sistema como una asignada por el usuario.

   Ejemplo:
   ```azurecli
   az vm identity assign -g <resource-group-name> -n <vm name>
   ```

1. Asigne el acceso de identidad administrada al grupo de recursos de la VM o a todas las interfaces de red, discos administrados y la propia VM, tal como se describe en [Asignación de un acceso de identidad administrada a un recurso mediante la CLI de Azure][howto-assign-access-cli].

    Ejemplo:

    ```azurecli
    spID=$(az resource show -g <resource-group-name> -n <vm name> --query identity.principalId --out tsv --resource-type Microsoft.Compute/virtualMachines)
    rgId=$(az group show -g <resource-group-name> --query id --out tsv)
    az role assignment create --assignee $spID --role 'Reader' --scope $rgId
    ```

1. Ejecute el siguiente comando de la CLI de Azure para instalar la extensión de Azure para SAP.
    Actualmente, la extensión solo se admite en AzureCloud. Azure China 21Vianet, Azure Government o cualquiera de los demás entornos especiales todavía no se admiten.

    ```azurecli
    # For Linux machines
    az vm extension set --publisher Microsoft.AzureCAT.AzureEnhancedMonitoring --name MonitorX64Linux --version 1.0 -g <resource-group-name> --vm-name <vm name> --settings '{"system":"SAP"}'

    #For Windows machines
    az vm extension set --publisher Microsoft.AzureCAT.AzureEnhancedMonitoring --name MonitorX64Windows --version 1.0 -g <resource-group-name> --vm-name <vm name> --settings '{"system":"SAP"}'
    ```

## <a name="checks-and-troubleshooting"></a><a name="564adb4f-5c95-4041-9616-6635e83a810b"></a>Comprobaciones y solución de problemas

Después de implementar la máquina virtual de Azure y de configurar la extensión de Azure para SAP correspondiente, compruebe si todos los componentes de la extensión funcionan según lo previsto.

Ejecute la comprobación de preparación de la extensión de Azure para SAP tal y como se describe en [Comprobación de la preparación de la extensión de Azure para SAP][deployment-guide-5.1]. Si todos los resultados de la comprobación de la preparación son positivos y todos los contadores de rendimiento pertinentes son correctos, la extensión de Azure para SAP se configuró correctamente. Puede continuar con la instalación de SAP Host Agent, tal y como se describe en las notas de SAP en [Recursos de SAP][deployment-guide-2.2]. Si la comprobación de la preparación indica que faltan contadores, ejecute la comprobación de estado de la extensión de Azure para SAP, tal como se describe en [Comprobación del estado de la configuración de la extensión de Azure para SAP][deployment-guide-5.2]. Para opciones de solución de problemas, consulte [Solución de problemas de la extensión de Azure para SAP][deployment-guide-5.3].

### <a name="readiness-check-for-the-azure-extension-for-sap"></a><a name="bb61ce92-8c5c-461f-8c53-39f5e5ed91f2"></a>Comprobación de la preparación de la extensión de Azure para SAP

> [!NOTE]
> Hay dos versiones de la extensión de VM. En este capítulo se trata la extensión de VM predeterminada. Si ha instalado la nueva extensión de VM, consulte el capítulo [Comprobación de la preparación de la nueva extensión de Azure para SAP][deployment-guide-5.1-new].

Con esta comprobación, se asegura de que la extensión de Azure para SAP subyacente proporcione todas las métricas que aparecen dentro de la aplicación SAP.

#### <a name="run-the-readiness-check-on-a-windows-vm"></a>Ejecución de la comprobación de la preparación en una máquina virtual Windows

1. Inicie sesión en la máquina virtual de Azure (no se necesita una cuenta de administrador).
1. Abra una ventana de símbolo del sistema.
1. En el símbolo del sistema, cambie el directorio a la carpeta de instalación de la extensión de Azure para SAP: C:\\Packages\\Plugins\\Microsoft.AzureCAT.AzureEnhancedMonitoring.AzureCATExtensionHandler\\&lt;versión>\\drop.

   La *versión* que aparece en la ruta de acceso a la extensión puede variar. Si ve carpetas de varias versiones de la extensión en la carpeta de instalación, revise la configuración del servicio de Windows AzureEnhancedMonitoring y cambie a la carpeta indicada como *Ruta de acceso al ejecutable*.

   ![Propiedades del servicio que ejecuta la extensión de Azure para SAP][deployment-guide-figure-1000]

1. En el símbolo del sistema, ejecute **azperflib.exe** sin ningún parámetro.

   > [!NOTE]
   > Azperflib.exe se ejecuta en un bucle y actualiza los contadores recopilados cada 60 segundos. Para finalizar el bucle, cierre la ventana de símbolo del sistema.
   >
   >

Si la extensión de Azure para SAP no está instalada, o el servicio AzureEnhancedMonitoring no se está ejecutando, la extensión no se configuró correctamente. Para información detallada sobre cómo implementar la extensión, consulte [Solución de problemas de la extensión de Azure para SAP][deployment-guide-5.3].

> [!NOTE]
> Azperflib.exe es un componente que no se puede usar para fines propios. Proporciona datos de la infraestructura de Azure relacionados con la VM para SAP Host Agent de manera exclusiva.
> 

##### <a name="check-the-output-of-azperflibexe"></a>Revise la salida del archivo azperflib.exe.

La salida del archivo azperflib.exe muestra todos los contadores de rendimiento de Azure para SAP rellenados. En la parte inferior de la lista de los contadores recopilados, encontrará un resumen y un indicador que muestra el estado de la extensión de Azure para SAP.

![Salida de la comprobación del estado mediante la ejecución de azperflib.exe, que indica que no hay problemas][deployment-guide-figure-1100]
<a name="figure-11"></a>

Vea el valor de **Counters total** (Total de contadores) en la salida, que se notifica como vacío, y el valor de **Health status** (Estado de mantenimiento), que se muestra en la ilustración anterior.

Puede interpretar los valores del resultado de la siguiente manera:

| Valores del resultado de azperflib.exe | Estado de mantenimiento de la extensión de Azure para SAP |
| --- | --- |
| **Llamadas a API: no disponible** | Los contadores que no están disponibles puede que no se utilicen con la configuración de las máquinas virtuales o que haya errores. Consulte **Estado de mantenimiento**. |
| **Total de contadores: vacíos** |Los siguientes dos contadores Azure Storage pueden estar vacíos: <ul><li>Storage Read Op Latency Server msec</li><li>Storage Read Op Latency E2E msec</li></ul>Todos los demás contadores deben tener valores. |
| **Estado de mantenimiento** |Solo es correcto si el estado del resultado muestra **Correcto**. |
| **Diagnóstico** |Información detallada sobre el estado de mantenimiento. |

Si el valor de **Estado de mantenimiento** no es **Correcto**, siga las instrucciones de [Comprobación del estado de la configuración de la extensión de Azure para SAP][deployment-guide-5.2].

#### <a name="run-the-readiness-check-on-a-linux-vm"></a>Ejecución de la comprobación de la preparación en una máquina virtual Linux

1. Conéctese a la máquina virtual de Azure mediante SSH.

1. Compruebe la salida de la extensión de Azure para SAP.

   a.  Ejecute `more /var/lib/AzureEnhancedMonitor/PerfCounters`:

   **Resultado esperado**: devuelve una lista de contadores de rendimiento. El archivo no debe estar vacío.

   b. Ejecute `cat /var/lib/AzureEnhancedMonitor/PerfCounters | grep Error`:

   **Resultado esperado**: devuelve una línea en la que el error es **none**, por ejemplo, **3;config;Error;;0;0;none;0;1456416792;tst-servercs;** .

   c. Ejecute `more /var/lib/AzureEnhancedMonitor/LatestErrorRecord`:

   **Resultado esperado**: devuelve un resultado vacío o no existente.

Si la comprobación anterior no fue correcta, ejecute estas comprobaciones adicionales:

1. Asegúrese de que waagent esté instalado y habilitado.

   a.  Ejecute `sudo ls -al /var/lib/waagent/`:

     **Resultado esperado**: muestra el contenido del directorio de waagent.

   b.  Ejecute `ps -ax | grep waagent`:

   **Resultado esperado**: muestra una entrada similar a: `python /usr/sbin/waagent -daemon`.

1. Asegúrese de que la extensión de Azure para SAP esté instalada y en ejecución.

   a.  Ejecute `sudo sh -c 'ls -al /var/lib/waagent/Microsoft.OSTCExtensions.AzureEnhancedMonitorForLinux-*/'`:

   **Resultado esperado**: muestra el contenido del directorio de la extensión de Azure para SAP.

   b. Ejecute `ps -ax | grep AzureEnhanced`:

   **Resultado esperado**: muestra una entrada similar a: `python /var/lib/waagent/Microsoft.OSTCExtensions.AzureEnhancedMonitorForLinux-2.0.0.2/handler.py daemon`.

1. Instale SAP Host Agent tal y como se describe en la nota de SAP [1031096], y compruebe la salida de `saposcol`.

   a.  Ejecute `/usr/sap/hostctrl/exe/saposcol -d`:

   b.  Ejecute `dump ccm`:

   c.  Compruebe si la métrica **Virtualization_Configuration\Enhanced Monitoring Access** es **true**.

Si ya tiene instalado un servidor de aplicaciones de SAP NetWeaver ABAP, abra la transacción ST06 y compruebe si la supervisión está habilitada.

Si se produce un error en alguna de estas comprobaciones o necesita información detallada acerca de cómo volver a implementar la extensión, consulte [Solución de problemas de la extensión de Azure para SAP][deployment-guide-5.3].

### <a name="readiness-check-for-the-new-azure-extension-for-sap"></a><a name="7bf24f59-7347-4c7a-b094-4693e4687ee5"></a>Comprobación de la preparación de la nueva extensión de Azure para SAP

> [!NOTE]
> Hay dos versiones de la extensión de VM. En este capítulo se trata la nueva extensión de VM. Si ha instalado la extensión de VM predeterminada, consulte el capítulo [Comprobación de la preparación de la extensión de Azure para SAP][deployment-guide-5.1].

Con esta comprobación, se asegura de que la extensión de Azure para SAP subyacente proporcione todas las métricas que aparecen dentro de la aplicación SAP.

#### <a name="run-the-readiness-check-on-a-windows-vm"></a>Ejecución de la comprobación de la preparación en una máquina virtual Windows

1. Inicie sesión en la máquina virtual de Azure (no se necesita una cuenta de administrador).
1. Abra un explorador web y vaya a http://127.0.0.1:11812/azure4sap/metrics.
1. El explorador debe mostrar o descargar un archivo XML con los datos de supervisión de la máquina virtual. Si no lo hace, asegúrese de que la extensión de Azure para SAP esté instalada.

##### <a name="check-the-content-of-the-xml-file"></a>Comprobación del contenido del archivo XML

El archivo XML al que se puede acceder desde http://127.0.0.1:11812/azure4sap/metrics contiene todos los contadores de rendimiento de Azure completados para SAP. También contiene un resumen y un indicador de mantenimiento del estado de la extensión de Azure para SAP.

Compruebe el valor del elemento **Provider Health Description** (Descripción de mantenimiento del proveedor). Si el valor no es **Correcto**, siga las instrucciones de [Comprobación del estado de la configuración de la extensión de Azure para SAP][deployment-guide-5.2-new].

#### <a name="run-the-readiness-check-on-a-linux-vm"></a>Ejecución de la comprobación de la preparación en una máquina virtual Linux

1. Conéctese a la máquina virtual de Azure mediante SSH.

1. Compruebe la salida del siguiente comando.

    ```console
    curl http://127.0.0.1:11812/azure4sap/metrics
    ```
    
   **Resultado esperado**: Devuelve un documento XML que contiene la información de supervisión de la máquina virtual, sus discos e interfaces de red.

Si la comprobación anterior no fue correcta, ejecute estas comprobaciones adicionales:

1. Asegúrese de que waagent esté instalado y habilitado.

   a.  Ejecute `sudo ls -al /var/lib/waagent/`:

     **Resultado esperado**: muestra el contenido del directorio de waagent.

   b.  Ejecute `ps -ax | grep waagent`:

   **Resultado esperado**: muestra una entrada similar a: `python /usr/sbin/waagent -daemon`.

1. Asegúrese de que la extensión de Azure para SAP esté instalada y en ejecución.

   a.  Ejecute `sudo sh -c 'ls -al /var/lib/waagent/Microsoft.AzureCAT.AzureEnhancedMonitoring.MonitorX64Linux-*/'`:

   **Resultado esperado**: muestra el contenido del directorio de la extensión de Azure para SAP.

   b. Ejecute `ps -ax | grep AzureEnhanced`:

   **Resultado esperado**: muestra una entrada similar a: `/var/lib/waagent/Microsoft.AzureCAT.AzureEnhancedMonitoring.MonitorX64Linux-1.0.0.82/AzureEnhancedMonitoring -monitor`.

1. Instale SAP Host Agent tal y como se describe en la nota de SAP [1031096], y compruebe la salida de `saposcol`.

   a.  Ejecute `/usr/sap/hostctrl/exe/saposcol -d`:

   b.  Ejecute `dump ccm`:

   c.  Compruebe si la métrica **Virtualization_Configuration\Enhanced Monitoring Access** es **true**.

Si ya tiene instalado un servidor de aplicaciones de SAP NetWeaver ABAP, abra la transacción ST06 y compruebe si la supervisión está habilitada.

Si se produce un error en alguna de estas comprobaciones o necesita información detallada acerca de cómo volver a implementar la extensión, consulte [Solución de problemas de la nueva extensión de Azure para SAP][deployment-guide-5.3-new].

### <a name="health-check-for-the-azure-extension-for-sap-configuration"></a><a name="e2d592ff-b4ea-4a53-a91a-e5521edb6cd1"></a>Comprobación del estado de la configuración de la extensión de Azure para SAP

> [!NOTE]
> Hay dos versiones de la extensión de VM. En este capítulo se trata la extensión de VM predeterminada. Si ha instalado la nueva extensión de VM, consulte el capítulo [Comprobación del estado de la configuración de la nueva extensión de Azure para SAP][deployment-guide-5.2-new].

Si parte de los datos de infraestructura no se entregan correctamente según lo indicado en la prueba que se describe en [Comprobación de la preparación de la extensión de Azure para SAP][deployment-guide-5.1], ejecute el cmdlet `Test-AzVMAEMExtension` para comprobar si la configuración de la infraestructura de Azure y la extensión de Azure para SAP es correcta.

1. Asegúrese de tener instalada la versión más reciente del cmdlet de Azure PowerShell, tal y como se describe en [Implementación de cmdlets de Azure PowerShell][deployment-guide-4.1].
1. Ejecute el siguiente cmdlet de PowerShell. Para ver la lista de entornos disponibles, ejecute el cmdlet `Get-AzEnvironment`. Para usar Azure global, seleccione el entorno **AzureCloud**. Para Azure China 21Vianet, seleccione **AzureChinaCloud**.

   ```powershell
   $env = Get-AzEnvironment -Name <name of the environment>
   Connect-AzAccount -Environment $env
   Set-AzContext -SubscriptionName <subscription name>
   Test-AzVMAEMExtension -ResourceGroupName <resource group name> -VMName <virtual machine name>
   ```

1. El script comprueba la configuración de la máquina virtual que seleccione.

   ![Salida de la prueba correcta de la extensión de Azure para SAP][deployment-guide-figure-1300]

Asegúrese de que todos los resultados de la comprobación del estado de mantenimiento sean **OK**. Si algunas comprobaciones no muestran **OK**, ejecute el cmdlet de actualización tal y como se describe en [Configuración de la extensión de Azure para SAP][deployment-guide-4.5]. Espere 15 minutos y repita las comprobaciones que se describen en [Comprobación de la preparación de la extensión de Azure para SAP][deployment-guide-5.1] y [Comprobación del estado de la configuración de la extensión de Azure para SAP][deployment-guide-5.2]. Si las comprobaciones siguen indicando problemas en algunos contadores o en todos ellos, consulte [Solución de problemas de la extensión de Azure para SAP][deployment-guide-5.3].

> [!Note]
> En casos en los que use Azure Managed Disks estándar, puede que experimente algunas advertencias. En lugar de que las pruebas devuelvan "Correcto", se mostrarán advertencias. Esto es normal y se espera en el caso de ese tipo de disco. Consulte también [Solución de problemas de la extensión de Azure para SAP][deployment-guide-5.3].
> 

### <a name="health-check-for-the-new-azure-extension-for-sap-configuration"></a><a name="464ac96d-7d3c-435d-a5ae-3faf3bfef4b3"></a>Comprobación del estado de la configuración de la nueva extensión de Azure para SAP

> [!NOTE]
> Hay dos versiones de la extensión de VM. En este capítulo se trata la nueva extensión de VM. Si ha instalado la extensión de VM predeterminada, consulte el capítulo [Comprobación del estado de la configuración de la extensión de Azure para SAP][deployment-guide-5.2].

Si parte de los datos de infraestructura no se entregan correctamente según lo indicado en la prueba que se describe en [Comprobación de la preparación de la extensión de Azure para SAP][deployment-guide-5.1-new], ejecute el cmdlet `Get-AzVMExtension` para comprobar si la extensión de Azure para SAP está instalada. `Test-AzVMAEMExtension` aún no admite la nueva extensión. Cuando el cmdlet admita la nueva extensión, actualizaremos este artículo.

1. Asegúrese de tener instalada la versión más reciente del cmdlet de Azure PowerShell, tal y como se describe en [Implementación de cmdlets de Azure PowerShell][deployment-guide-4.1].
1. Ejecute el siguiente cmdlet de PowerShell. Para ver la lista de entornos disponibles, ejecute el cmdlet `Get-AzEnvironment`. Para usar Azure global, seleccione el entorno **AzureCloud**. Para Azure China 21Vianet, seleccione **AzureChinaCloud**.

   ```powershell
   $env = Get-AzEnvironment -Name <name of the environment>
   Connect-AzAccount -Environment $env
   Set-AzContext -SubscriptionName <subscription name>
   Test-AzVMAEMExtension -ResourceGroupName <resource group name> -VMName <virtual machine name>
   ```

1. El cmdlet comprueba la configuración de la extensión de VM para SAP en la máquina virtual que seleccione.

### <a name="troubleshooting-azure-extension-for-sap"></a><a name="fe25a7da-4e4e-4388-8907-8abc2d33cfd8"></a>Solución de problemas de la extensión de Azure para SAP

> [!NOTE]
> Hay dos versiones de la extensión de VM. En este capítulo se trata la extensión de VM predeterminada. Si ha instalado la nueva extensión de VM, consulte el capítulo [Solución de problemas de la extensión de Azure para SAP][deployment-guide-5.3-new].

#### <a name="windowslogo_windows-azure-performance-counters-do-not-show-up-at-all"></a>![Windows][Logo_Windows] No aparecen contadores de rendimiento de Azure

El servicio AzureEnhancedMonitoring Windows recopila métricas de rendimiento en Azure. Si el servicio no se instaló correctamente o si no se está ejecutando en la máquina virtual, no se recopilará ninguna métrica de rendimiento.

##### <a name="the-installation-directory-of-the-azure-extension-for-sap-is-empty"></a>El directorio de instalación de la extensión de Azure para SAP está vacío

###### <a name="issue"></a>Incidencia

El directorio de instalación C:\\Packages\\Plugins\\Microsoft.AzureCAT.AzureEnhancedMonitoring.AzureCATExtensionHandler\\&lt;versión>\\drop está vacío.

###### <a name="solution"></a>Solución

La extensión no está instalada. Determine si se trata de un problema de proxy (como se describió anteriormente). Puede que tenga que reiniciar la máquina o volver a ejecutar el script de configuración `Set-AzVMAEMExtension`.

##### <a name="service-for-azure-extension-for-sap-does-not-exist"></a>No existe el servicio para la extensión de Azure para SAP

###### <a name="issue"></a>Incidencia

El servicio de Windows AzureEnhancedMonitoring no existe.

La salida de Azperflib.exe produce un error:

![La ejecución de azperflib.exe indica que el servicio de la extensión de Azure para SAP no está en ejecución][deployment-guide-figure-1400]
<a name="figure-14"></a>

###### <a name="solution"></a>Solución

Si el servicio no existe, significa que la extensión de Azure para SAP no se instaló correctamente. Implemente de nuevo la extensión siguiendo los pasos del escenario de implementación correspondiente que se indican en [Escenarios de implementación de máquinas virtuales para SAP en Azure][deployment-guide-3].

Después de implementar la extensión, vuelva a comprobar en una hora si los contadores de rendimiento de Azure aparecen en la máquina virtual de Azure.

##### <a name="service-for-azure-extension-for-sap-exists-but-fails-to-start"></a>El servicio de la extensión de Azure para SAP existe, pero no se inicia

###### <a name="issue"></a>Incidencia

El servicio de Windows AzureEnhancedMonitoring existe y está habilitado, pero no se inicia. Consulte el registro de eventos de la aplicación para más información.

###### <a name="solution"></a>Solución

La configuración es incorrecta. Reinicie la extensión de Azure para SAP en la máquina virtual, tal como se describe en [Configuración de la extensión de Azure para SAP][deployment-guide-4.5].

#### <a name="windowslogo_windows-some-azure-performance-counters-are-missing"></a>![Windows][Logo_Windows] Faltan algunos contadores de rendimiento de Azure

El servicio AzureEnhancedMonitoring Windows recopila métricas de rendimiento en Azure. El servicio obtiene datos de varios orígenes. Algunos datos de configuración se recopilan localmente, y algunas métricas de rendimiento se leen de Diagnósticos de Azure. Los contadores de almacenamiento proceden de los registros en el nivel de suscripción de almacenamiento.

Si la solución de problemas descrita en la nota de SAP [1999351] no resuelve el problema, vuelva a ejecutar el script de configuración `Set-AzVMAEMExtension`. Puede que tenga que esperar una hora porque los contadores de diagnóstico o análisis de almacenamiento no se crean de inmediato una vez habilitados. Si el problema continúa, abra un mensaje de servicio al cliente de SAP acerca del componente BC-OP-NT-AZR para una máquina virtual Windows o BC-OP-LNX-AZR para una máquina virtual Linux.

#### <a name="linuxlogo_linux-azure-performance-counters-do-not-show-up-at-all"></a>![Linux][Logo_Linux] No aparecen contadores de rendimiento de Azure

Un demonio recopila las métricas de rendimiento en Azure. Si el demonio no se está ejecutando, no se recopilará ninguna métrica de rendimiento.

##### <a name="the-installation-directory-of-the-azure-extension-for-sap-is-empty"></a>El directorio de instalación de la extensión de Azure para SAP está vacío

###### <a name="issue"></a>Incidencia

El directorio \\var\\lib\\waagent\\ no contiene un subdirectorio para la extensión de Azure para SAP.

###### <a name="solution"></a>Solución

La extensión no está instalada. Determine si se trata de un problema de proxy (como se describió anteriormente). Puede que tenga que reiniciar la máquina o volver a ejecutar el script de configuración `Set-AzVMAEMExtension`.

##### <a name="the-execution-of-set-azvmaemextension-and-test-azvmaemextension-show-warning-messages-stating-that-standard-managed-disks-are-not-supported"></a>La ejecución de Set-AzVMAEMExtension y Test-AzVMAEMExtension muestra mensajes de advertencia que indican que Managed Disks estándar no se admite.

###### <a name="issue"></a>Incidencia

Al ejecutar Set-AzVMAEMExtension o Test-AzVMAEMExtension, se muestran mensajes como los siguientes:

<pre><code>
WARNING: [WARN] Standard Managed Disks are not supported. Extension will be installed but no disk metrics will be available.
WARNING: [WARN] Standard Managed Disks are not supported. Extension will be installed but no disk metrics will be available.
WARNING: [WARN] Standard Managed Disks are not supported. Extension will be installed but no disk metrics will be available.
</code></pre>

Cuando ejecuta azperfli.exe como se describió anteriormente, puede obtener un resultado que indica un estado incorrecto. 

###### <a name="solution"></a>Solución

Los mensajes se deben a que el servicio Managed Disks estándar no proporciona las API que usa la extensión de Azure para SAP para comprobar las estadísticas de las cuentas de Azure Storage estándar. Esto no es motivo de preocupación. La razón para introducir la recopilación de datos de las cuentas de almacenamiento de discos estándar era limitar las entradas y salidas que se producían con frecuencia. Los discos administrados evitan tal limitación al restringir el número de discos en una cuenta de almacenamiento. Por lo tanto, no es esencial tener ese tipo de datos.


#### <a name="linuxlogo_linux-some-azure-performance-counters-are-missing"></a>![Linux][Logo_Linux] Faltan algunos contadores de rendimiento de Azure

Un demonio recopila las métricas de rendimiento en Azure, y obtiene datos de varios orígenes. Algunos datos de configuración se recopilan localmente, y algunas métricas de rendimiento se leen de Diagnósticos de Azure. Los contadores de almacenamiento proceden de los registros en su suscripción de almacenamiento.

Para ver una lista completa y actualizada de los problemas conocidos, consulte la nota de SAP [1999351], que contiene información adicional sobre la solución de problemas de la extensión de Azure para SAP.

Si la solución de problemas descrita en la nota de SAP [1999351] no resuelve el problema, vuelva a ejecutar el script de configuración `Set-AzVMAEMExtension` tal y como se describe en [Configuración de la extensión de Azure para SAP][deployment-guide-4.5]. Puede que tenga que esperar una hora porque los contadores de diagnóstico o análisis de almacenamiento no se crean de inmediato una vez habilitados. Si el problema continúa, abra un mensaje de servicio al cliente de SAP acerca del componente BC-OP-NT-AZR para una máquina virtual Windows o BC-OP-LNX-AZR para una máquina virtual Linux.

### <a name="troubleshooting-the-new-azure-extension-for-sap"></a><a name="b7afb8ef-a64c-495d-bb37-2af96688c530"></a>Solución de problemas de la nueva extensión de Azure para SAP

> [!NOTE]
> Hay dos versiones de la extensión de VM. En este capítulo se trata la nueva extensión de VM. Si ha instalado la extensión de VM predeterminada, consulte el capítulo [Solución de problemas de la extensión de Azure para SAP][deployment-guide-5.3].

#### <a name="windowslogo_windows-azure-performance-counters-do-not-show-up-at-all"></a>![Windows][Logo_Windows] No aparecen contadores de rendimiento de Azure

El proceso AzureEnhancedMonitoring recopila métricas de rendimiento en Azure. Si el proceso no se está ejecutando en la VM, no se recopilará ninguna métrica de rendimiento.

##### <a name="the-installation-directory-of-the-azure-extension-for-sap-is-empty"></a>El directorio de instalación de la extensión de Azure para SAP está vacío

###### <a name="issue"></a>Incidencia

El directorio de instalación C:\\Packages\\Plugins\\Microsoft.AzureCAT.AzureEnhancedMonitoring.MonitorX64Windows\\&lt;versión> está vacío.

###### <a name="solution"></a>Solución

La extensión no está instalada. Determine si se trata de un problema de proxy (como se describió anteriormente). Es posible que tenga que reiniciar la máquina o volver a instalar la extensión de VM.

#### <a name="windowslogo_windows-some-azure-performance-counters-are-missing"></a>![Windows][Logo_Windows] Faltan algunos contadores de rendimiento de Azure

El proceso AzureEnhancedMonitoring de Windows recopila métricas de rendimiento en Azure. El proceso obtiene datos de varios orígenes. Algunos datos de configuración se recopilan localmente, y algunas métricas de rendimiento se leen de Azure Monitor.

Si la solución de problemas de la nota de SAP [1999351] no resuelve el problema, abra un mensaje de servicio al cliente de SAP acerca del componente BC-OP-NT-AZR para Windows o BC-OP-LNX-AZR para una máquina virtual Linux. Adjunte el archivo de registro C:\\Packages\\Plugins\\Microsoft.AzureCAT.AzureEnhancedMonitoring.MonitorX64Windows\\&lt;versión>\\logapp.txt al incidente.

#### <a name="linuxlogo_linux-azure-performance-counters-do-not-show-up-at-all"></a>![Linux][Logo_Linux] No aparecen contadores de rendimiento de Azure

Un demonio recopila las métricas de rendimiento en Azure. Si el demonio no se está ejecutando, no se recopilará ninguna métrica de rendimiento.

##### <a name="the-installation-directory-of-the-azure-extension-for-sap-is-empty"></a>El directorio de instalación de la extensión de Azure para SAP está vacío

###### <a name="issue"></a>Incidencia

El directorio \\var\\lib\\waagent\\ no contiene un subdirectorio para la extensión de Azure para SAP.

###### <a name="solution"></a>Solución

La extensión no está instalada. Determine si se trata de un problema de proxy (como se describió anteriormente). Es posible que tenga que reiniciar la máquina o volver a instalar la extensión de VM.

#### <a name="linuxlogo_linux-some-azure-performance-counters-are-missing"></a>![Linux][Logo_Linux] Faltan algunos contadores de rendimiento de Azure

Un demonio recopila las métricas de rendimiento en Azure, y obtiene datos de varios orígenes. Algunos datos de configuración se recopilan localmente y algunas métricas de rendimiento se leen de Azure Monitor.

Para ver una lista completa y actualizada de los problemas conocidos, consulte la nota de SAP [1999351], que contiene información adicional sobre la solución de problemas de la extensión de Azure para SAP.

Si la solución de problemas descrita en la nota de SAP [1999351] no resuelve el problema, vuelva a instalar la extensión tal y como se describe en [Configuración de la extensión de Azure para SAP][deployment-guide-4.5]. Si el problema continúa, abra un mensaje de servicio al cliente de SAP acerca del componente BC-OP-NT-AZR para una máquina virtual Windows o BC-OP-LNX-AZR para una máquina virtual Linux. Adjunte el archivo de registro /var/lib/waagent/Microsoft.AzureCAT.AzureEnhancedMonitoring.MonitorX64Linux-&lt;version>/logapp.txt al incidente.

## <a name="azure-extension-error-codes"></a>Códigos de error de extensión de Azure

| Id. de error | Descripción del error | Solución |
|---|---|---|
| <a name="cfg_018"></a>cfg/018 | Falta la configuración de la aplicación. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="cfg_019"></a>cfg/019 | No hay ningún identificador de implementación en la configuración de la aplicación. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="cfg_020"></a>cfg/020 | No hay ningún valor de RoleInstanceId en la configuración de la aplicación. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="cfg_022"></a>cfg/022 | No hay ningún valor de RoleInstanceId en la configuración de la aplicación. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="cfg_031"></a>cfg/031 | No se puede leer la configuración de Azure. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="cfg_021"></a>cfg/021 | Falta el archivo de configuración de la aplicación. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="cfg_015"></a>cfg/015 | No hay ningún tamaño de VM en la configuración de la aplicación. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="cfg_016"></a>cfg/016 | Error del contador GlobalMemoryStatusEx. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="cfg_023"></a>cfg/023 | Error del contador MaxHwFrequency. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="cfg_024"></a>cfg/024 | Error en los contadores de NIC. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="cfg_025"></a>cfg/025 | Error en el contador de asignación de disco. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="cfg_026"></a>cfg/026 | Error del contador del nombre del procesador. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="cfg_027"></a>cfg/027 | Error en el contador de asignación de disco. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="cfg_038"></a>cfg/038 | Falta la métrica "Disk type" en el archivo de configuración de la extensión config.xml. "Disk type", junto con otros contadores, se introdujo en la versión 2.2.0.68 el 16/12/2015. Si implementó la extensión antes de 16/12/2015, usa el archivo de configuración anterior. El marco de la extensión de Azure actualiza automáticamente la extensión a una versión más reciente, pero el archivo config.xml permanece inalterado. Para actualizar la configuración, descargue y ejecute el script de configuración de PowerShell más reciente. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="cfg_039"></a>cfg/039 | Sin almacenamiento en caché de disco. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="cfg_036"></a>cfg/036 | Sin rendimiento de SLA de disco. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="cfg_037"></a>cfg/037 | Sin IOPS de SLA de disco. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="cfg_028"></a>cfg/028 | Error en el contador de asignación de disco. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="cfg_029"></a>cfg/029 | Error del contador de cambios más recientes de hardware. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="cfg_030"></a>cfg/030 | Error en los contadores de NIC. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="cfg_017"></a>cfg/017 | Debido a sysprep en la VM, el SID de Windows ha cambiado. | [repita la implementación después de sysprep][deployment-guide-redeploy-after-sysprep] |
| <a name="str_007"></a>str/007 | Error de acceso al análisis de almacenamiento. <br /><br />Dado que el rellenado de los datos de análisis de almacenamiento en una VM recién creada puede necesitar hasta media hora, el error puede desaparecer después de un tiempo. El error sigue apareciendo, vuelva a ejecutar el script de instalación. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="str_010"></a>str/010 | No hay contadores de Storage Analytics. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="str_009"></a>str/009 | Error de Storage Analytics. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="wad_004"></a>wad/004 | Configuración de WAD incorrecta. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="wad_002"></a>wad/002 | Formato de WAD inesperado. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="wad_001"></a>wad/001 | No se encontró ningún contador de WAD. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="wad_040"></a>wad/040 | No se encontró ningún contador de WAD. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="wad_003"></a>wad/003 | No se puede leer la tabla de WAD. No existe ninguna conexión con la tabla de WAD. Puede haber varias causas:<br /><br /> 1) configuración obsoleta <br />2) no hay conexión de red con Azure <br />3) problemas con la configuración de WAD | [ejecute el script de instalación][deployment-guide-run-the-script]<br />[repare la conexión a Internet][deployment-guide-fix-internet-connection]<br />[póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="prf_011"></a>prf/011 | Error en las métricas de NIC de Perfmon. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="prf_012"></a>prf/012 | Error en las métricas de disco de Perfmon. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="prf_013"></a>prf/013 | Error en algunas métricas de prefmon. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="prf_014"></a>prf/014 | Error de Perfmon al crear un contador. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="cfg_035"></a>cfg/035 | No hay proveedores de métricas configurados. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |
| <a name="str_006"></a>str/006 | Configuración de Storage Analytics incorrecta. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="str_032"></a>str/032 | Error en las métricas de Storage Analytics. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="cfg_033"></a>cfg/033 | Error en uno de los proveedores de métricas. | [ejecute el script de instalación][deployment-guide-run-the-script] |
| <a name="str_034"></a>str/034 | Error en un subproceso del proveedor. | [póngase en contacto con el servicio de soporte técnico][deployment-guide-contact-support] |

### <a name="detailed-guidelines-on-solutions-provided"></a>Instrucciones detalladas sobre las soluciones proporcionadas

#### <a name="run-the-setup-script"></a><a name="0d2847ad-865d-4a4c-a405-f9b7baaa00c7"></a>Ejecute el script de configuración

Siga los pasos descritos en el capítulo [Configuración de la extensión de Azure para SAP][deployment-guide-4.5] de esta guía para volver a instalar la extensión. Tenga en cuenta que algunos contadores pueden necesitar hasta 30 minutos para el aprovisionamiento.

Si los errores no desaparecen, [póngase en contacto con el soporte técnico][deployment-guide-contact-support].

#### <a name="contact-support"></a><a name="3ba34cfc-c9bb-4648-9c3c-88e8b9130ca2"></a>Póngase en contacto con el soporte técnico

Hay un error inesperado o no hay ninguna solución conocida. Recopile el archivo AzureEnhancedMonitoring_service.log ubicado en la carpeta C:\Packages\Plugins\Microsoft.AzureCAT.AzureEnhancedMonitoring.AzureCATExtensionHandler\\\<version\>\drop (Windows) o /var/log/azure/Microsoft.OSTCExtensions.AzureEnhancedMonitorForLinux (Linux) y póngase en contacto con el soporte técnico de SAP para recibir más ayuda.

#### <a name="redeploy-after-sysprep"></a><a name="2cd61f22-187d-42ed-bb8c-def0c983d756"></a>Repita la implementación después de sysprep

Si tiene previsto compilar una imagen de SO de sysprep generalizada (que puede incluir software de SAP), se recomienda que esta imagen no incluya la extensión de Azure para SAP. Debe instalar la extensión de Azure para SAP una vez implementada la nueva instancia de la imagen de sistema operativo generalizada.

Sin embargo, si la imagen de sistema operativo generalizada y preparada con sysprep ya contiene la extensión de Azure para SAP, puede aplicar la siguiente solución alternativa para volver a configurar la extensión en la instancia de VM recién implementada:

* En la instancia de VM recién implementada, elimine el contenido de las siguientes carpetas:  
  C:\Packages\Plugins\Microsoft.AzureCAT.AzureEnhancedMonitoring.AzureCATExtensionHandler\\\<version\>\RuntimeSettings C:\Packages\Plugins\Microsoft.AzureCAT.AzureEnhancedMonitoring.AzureCATExtensionHandler\\\<version\>\Status

* Siga los pasos descritos en el capítulo [Configuración de la extensión de Azure para SAP][deployment-guide-4.5] de esta guía para volver a instalar la extensión.

#### <a name="fix-internet-connection"></a><a name="e92bc57d-80d9-4a2b-a2f4-16713a22ad89"></a>Repare la conexión a Internet

La máquina virtual de Microsoft Azure que ejecuta la extensión de Azure para SAP requiere acceso a Internet. Si esta VM de Azure forma parte de una red virtual de Azure o de un dominio local, asegúrese de que la configuración de proxy correspondiente esté establecida. Esta configuración también debe ser válida para que la cuenta LocalSystem tenga acceso a Internet. Siga el capítulo [Configuración del proxy][deployment-guide-configure-proxy] de esta guía.

Además, si necesita establecer una dirección IP estática para la VM de Azure, no la configure manualmente dentro de la VM de Azure, sino mediante [Azure PowerShell](../../../virtual-network/virtual-networks-static-private-ip-arm-ps.md), la [CLI de Azure](../../../virtual-network/virtual-networks-static-private-ip-arm-cli.md), [Azure Portal](../../../virtual-network/virtual-networks-static-private-ip-arm-pportal.md). La dirección IP estática se propaga mediante el servicio DHCP de Azure.

No se admite la definición manual de una dirección IP estática dentro de la VM de Azure y puede provocar problemas con la extensión de Azure para SAP.
