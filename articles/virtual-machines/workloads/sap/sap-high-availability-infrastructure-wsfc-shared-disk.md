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
ms.date: 05/05/2017
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: e682232afa401f443ffe8f14f617b075978117ea
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080053"
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


> ![Windows][Logo_Windows] Windows
>

Este artículo describe los pasos a seguir para preparar la infraestructura de Azure para instalar y configurar un sistema SAP de alta disponibilidad en un clúster de conmutación por error de Windows mediante el uso de un *disco compartido de clúster* como una opción de agrupación en clústeres de una instancia de ASCS de SAP.

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar la instalación, consulte este artículo:

* [Guía de arquitectura: Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un disco compartido de clúster][sap-high-availability-guide-wsfc-shared-disk]

## <a name="prepare-the-infrastructure-for-architectural-template-1"></a>Preparación de la infraestructura de la plantilla 1 de arquitectura
Las plantillas de Azure Resource Manager para SAP ayudan a simplificar la implementación de los recursos necesarios.

Las plantillas de tres niveles de Azure Resource Manager también admiten escenarios de alta disponibilidad. Por ejemplo, la Plantilla 1 de arquitectura tiene dos clústeres. Cada clúster es un único punto de error de SAP para ASCS/SCS de SAP y DBMS.

Aquí puede obtener las plantillas de Azure Resource Manager para el escenario de ejemplo que se describe en este artículo:

* [Imagen de Azure Marketplace](https://github.com/Azure/azure-quickstart-templates/)  
* [Imagen de Azure Marketplace con Azure Managed Disks](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md)  
* [Imagen personalizada](https://github.com/Azure/azure-quickstart-templates/)
* [Imagen personalizada con Managed Disks](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-md)

Para preparar la plantilla 1 de arquitectura, siga estos pasos:

- En Azure Portal, en el panel **Parámetros**, en el cuadro **SYSTEMAVAILABILITY**, seleccione **Alta disponibilidad**.

  ![Figura 1: Configuración de los parámetros de Azure Resource Manager para alta disponibilidad de SAP][sap-ha-guide-figure-3000]

_**Ilustración 1:** Configuración de los parámetros de Azure Resource Manager para alta disponibilidad de SAP_


  Las plantillas crean:

  * **Máquinas virtuales**:
    * Máquinas virtuales del servidor de aplicaciones de SAP: \<SAPSystemSID\>-di-\<Number\>
    * Máquinas virtuales del clúster de ASCS/SCS: \<SAPSystemSID\>-ascs-\<Number\>
    * Clúster de DBMS: \<SAPSystemSID\>-db-\<Number\>

  * **Tarjetas de red para todas las máquinas virtuales con direcciones IP asociadas**:
    * \<SAPSystemSID\>-nic-di-\<Number\>
    * \<SAPSystemSID\>-nic-ascs-\<Number\>
    * \<SAPSystemSID\>-nic-db-\<Number\>

  * **Cuentas de Azure Storage (solo discos no administrados)** :

  * **Grupos de disponibilidad** para:
    * Máquinas virtuales del servidor de aplicaciones de SAP: \<SAPSystemSID\>-avset-di
    * Máquinas virtuales del clúster de ASCS/SCS de SAP: \<SAPSystemSID\>-avset-ascs
    * Máquinas virtuales del clúster de DBMS: \<SAPSystemSID\>-avset-db

  * **Equilibrador de carga interno de Azure**:
    * Con todos los puertos para la dirección IP y la instancia de ASCS/SCS \<SAPSystemSID\>-lb-ascs
    * Con todos los puertos para la dirección IP y DBMS de SQL Server \<SAPSystemSID\>-lb-db

  * **Grupo de seguridad de red**: \<SAPSystemSID\>-nsg-ascs-0  
    * Con un puerto de Protocolo de escritorio remoto (RDP) externo abierto para la máquina virtual \<SAPSystemSID\>-ascs-0

> [!NOTE]
> De manera predeterminada, todas las direcciones IP de las tarjetas de red y de los equilibradores de carga internos de Azure son dinámicas. Cámbielas a direcciones IP estáticas. Esto se describe más adelante en el artículo.
>
>

## <a name="deploy-virtual-machines-with-corporate-network-connectivity-cross-premises-to-use-in-production"></a><a name="c87a8d3f-b1dc-4d2f-b23c-da4b72977489"></a> Implementación de máquinas virtuales con conectividad de red corporativa (entre locales) para uso en producción
En los sistemas de producción de SAP, implemente máquinas virtuales de Azure con conectividad de red corporativa mediante Azure VPN Gateway o Azure ExpressRoute.

> [!NOTE]
> Puede usar la instancia de Azure Virtual Network. La red virtual y la subred ya se crearon y están preparadas.
>
>

1. En Azure Portal, en el panel **Parámetros**, en el cuadro **NEWOREXISTINGSUBNET**, seleccione **Existente**.
2. En el cuadro **SUBNETID**, agregue la cadena completa del identificador de subred de la red de Azure preparada donde planea implementar las máquinas virtuales de Azure.
3. Ejecute este comando de PowerShell para obtener una lista de todas las subredes de red de Azure:

   ```powershell
   (Get-AzVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets
   ```

   El campo **ID** muestra el valor del identificador de subred.
4. Ejecute este comando de PowerShell para obtener una lista de todos los valores de identificador de subred:

   ```powershell
   (Get-AzVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets.Id
   ```

   El aspecto del identificador de subred es similar al siguiente:

   ```
   /subscriptions/<subscription ID>/resourceGroups/<VPN name>/providers/Microsoft.Network/virtualNetworks/azureVnet/subnets/<subnet name>
   ```

## <a name="deploy-cloud-only-sap-instances-for-test-and-demo"></a><a name="7fe9af0e-3cce-495b-a5ec-dcb4d8e0a310"></a> Implementación solo en la nube de instancias de SAP para prueba o demostración
Puede implementar el sistema de SAP de alta disponibilidad en un modelo de implementación solo en la nube. Este tipo de implementación se usa principalmente para demostrar o probar casos de uso. No es idóneo para los casos de uso de producción.

- En Azure Portal, en el panel **Parámetros**, en el cuadro **NEWOREXISTINGSUBNET**, seleccione **Nuevo**. Deje vacío el campo **SUBNETID**.

  La plantilla de Azure Resource Manager para SAP crea automáticamente la subred y red virtual de Azure.

> [!NOTE]
> También debe implementar, como mínimo, una máquina virtual dedicada para Active Directory y el servicio DNS en la misma instancia de Azure Virtual Network. La plantilla no crea estas máquinas virtuales.
>
>


## <a name="prepare-the-infrastructure-for-architectural-template-2"></a>Preparación de la infraestructura de la plantilla 2 de arquitectura

Puede usar estas plantillas de Azure Resource Manager para SAP para simplificar la implementación de los recursos de infraestructura necesarios para la plantilla 2 de arquitectura de SAP.

Aquí puede obtener las plantillas de Azure Resource Manager para este escenario de implementación:

* [Imagen de Azure Marketplace](https://github.com/Azure/azure-quickstart-templates/)  
* [Imagen de Azure Marketplace con Managed Disks](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-converged-md)  
* [Imagen personalizada](https://github.com/Azure/azure-quickstart-templates/)
* [Imagen personalizada con Managed Disks](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-converged-md)


## <a name="prepare-the-infrastructure-for-architectural-template-3"></a>Preparación de la infraestructura de la plantilla 3 de arquitectura

Puede preparar la infraestructura y configurar SAP para varios SID. Por ejemplo, puede agregar una instancia de ASCS/SCS de SAP adicional en una configuración de clúster *existente*. Para obtener más información, vea [Configuración de una instancia adicional de ASCS/SCS de SAP en una configuración de clúster existente para crear una configuración de varios SID de SAP en Azure Resource Manager][sap-ha-multi-sid-guide].

Si desea crear un nuevo clúster de varios SID, puede usar las [plantillas de inicio rápido en GitHub](https://github.com/Azure/azure-quickstart-templates) para varios SID.

Para crear un nuevo clúster de varios SID, debe implementar las tres plantillas siguientes:

* [Plantilla de ASCS/SCS](#ASCS-SCS-template)
* [Plantilla de base de datos](#database-template)
* [Plantilla de servidores de aplicaciones](#application-servers-template)

Las siguientes secciones contienen más detalles sobre las plantillas y los parámetros que debe proporcionar en las plantillas.

### <a name="ascsscs-template"></a><a name="ASCS-SCS-template"></a>Plantilla de ASCS/SCS

La plantilla de ASCS/SCS permite implementar dos máquinas virtuales que se pueden usar para crear un clúster de conmutación por error de Windows Server que hospeda varias instancias de ASCS/SCS.

Para configurar la plantilla de varios SID de ASCS/SCS, en la [plantilla de varios SID de ASCS/SCS][sap-templates-3-tier-multisid-xscs-marketplace-image] o en la [plantilla de varios SID de ASCS/SCS con Managed Disks][sap-templates-3-tier-multisid-xscs-marketplace-image-md], especifique los valores para los parámetros siguientes:

- **Prefijo de recurso**:  Establezca el prefijo de recurso, que se utiliza para prefijar todos los recursos que se hayan creado durante la implementación. Dado que los recursos no pertenecen a un único sistema SAP, el prefijo del recurso no es el SID de un sistema SAP.  El prefijo debe tener entre tres y seis caracteres.
- **Tipo de pila**: Seleccione el tipo de la pila del sistema SAP. Dependiendo del tipo de la pila, Azure Load Balancer tendrá una (solo ABAP o JAVA) o dos (ABAP + JAVA) direcciones IP privadas por cada sistema SAP.
- **OS Type** (Tipo de sistema operativo): Seleccione el sistema operativo de las máquinas virtuales.
- **Recuento de sistema SAP**: seleccione el número de sistemas SAP que quiere instalar en este clúster.
- **Disponibilidad del sistema**: Seleccione **Alta disponibilidad**.
- **Nombre de usuario y contraseña del administrador**: Cree un nuevo usuario que pueda usarse para iniciar sesión en la máquina.
- **Subred nueva o existente**: determine si es necesario crear una red virtual y una subred o si se debe usar una subred ya existente. Si ya tiene una red virtual conectada a la red local, seleccione la **existente**.
- **Identificador de subred**: Si quiere implementar la máquina virtual en una red virtual existente en la que tiene una subred definida a la que se debe asignar la máquina virtual, asigne un nombre al identificador de esa subred específica. Normalmente, el identificador tiene este aspecto:

  /subscriptions/\<subscription id\>/resourceGroups/\<resource group name\>/providers/Microsoft.Network/virtualNetworks/\<virtual network name\>/subnets/\<subnet name\>

La plantilla implementa una instancia de Azure Load Balancer que admite varios sistemas SAP:

- Las instancias de ASCS se configuran según el número de instancia 00, 10, 20...
- Las instancias de SCS se configuran según el número de instancia 01, 11, 21...
- Las instancias de ASCS Enqueue Replication Server (ERS) (solo Linux) se configuran según el número de instancia 02, 12, 22, etc.
- Las instancias de SCS ERS (solo Linux) se configuran según el número de instancia 03, 13, 23...

El equilibrador de carga contiene 1 VIP (2 para Linux), 1 VIP para ASCS/SCS y 1 VIP para ERS (solo Linux).

#### <a name="sap-ascsscs-ports"></a><a name="0f3ee255-b31e-4b8a-a95a-d9ed6200468b"></a>Puertos de ASCS/SCS de SAP
En la lista siguiente se incluyen todas las reglas de equilibrio de carga (donde x es el número del sistema SAP, por ejemplo, 1, 2, 3, etc.):
- Puertos específicos de Windows para cada sistema SAP: 445, 5985
- Puertos ASCS (número de instancia x0): 32x0, 36x0, 39x0, 81x0, 5x013, 5x014, 5x016
- Puertos SCS (número de instancia x1): 32x1, 33x1, 39x1, 81x1, 5x113, 5x114, 5x116
- Puertos ASCS ERS en Linux (número de instancia x2): 33x2, 5x213, 5x214, 5x216
- Puertos SCS ERS en Linux (número de instancia x3): 33x3, 5x313, 5x314, 5x316

El equilibrador de carga se configurará para usar los siguientes puertos de sondeo (donde x es el número del sistema SAP, por ejemplo, 1, 2, 3,...):
- Puerto de sondeo del equilibrador de carga interno de ASCS/SCS: 620x0
- Puerto de sondeo del equilibrador de carga interno de ERS (solo para Linux): 621x2

### <a name="database-template"></a><a name="database-template"></a> Plantilla de base de datos

La plantilla de la base de datos implementa uno o dos máquinas virtuales que puede usar para instalar el sistema de administración de bases de datos relacionales (RDBMS) de un sistema SAP. Por ejemplo, si ha implementado una plantilla de ASCS/SCS para 5 sistemas SAP, debe implementar esta plantilla cinco veces.

Para configurar la plantilla de varios SID de base de datos, en la [plantilla de varios SID de base de datos][sap-templates-3-tier-multisid-db-marketplace-image] o en la [plantilla de varios SID de base de datos con Managed Disks][sap-templates-3-tier-multisid-db-marketplace-image-md], especifique los valores para los parámetros siguientes:

- **Identificador de sistema SAP**: escriba el identificador del sistema SAP que se va a instalar. El identificador se usa como prefijo de los recursos que se implementan.
- **Tipo de SO**: Seleccione el sistema operativo de las máquinas virtuales.
- **Dbtype**: seleccione el tipo de base de datos que quiere instalar en el clúster. Seleccione **SQL** si desea instalar Microsoft SQL Server. Seleccione **HANA** si tiene previsto instalar SAP HANA en las máquinas virtuales. Asegúrese de que selecciona el tipo de sistema operativo correcto. Seleccione **Windows** para SQL y seleccione una distribución de Linux para HANA. La instancia de Azure Load Balancer que está conectada a las máquinas virtuales está configurada para admitir el tipo de base de datos seleccionado:
  * **SQL**: El equilibrador de carga equilibra la carga del puerto 1433. Asegúrese de que utiliza este puerto para la configuración de SQL Server Always On.
  * **HANA**: El equilibrador de carga equilibra la carga de los puertos 35015 y 35017. Asegúrese de instalar SAP HANA con el número de instancia **50**.
  El equilibrador de carga usa el puerto de sondeo 62550.
- **Tamaño del sistema SAP**: Establezca la cantidad de SAPS que el sistema nuevo proporciona. Si no está seguro de cuántos SAPS necesita el sistema, consulte con el integrador de sistemas o el asociado tecnológico de SAP.
- **Disponibilidad del sistema**: Seleccione **Alta disponibilidad**.
- **Nombre de usuario y contraseña del administrador**: Cree un nuevo usuario que pueda usarse para iniciar sesión en la máquina.
- **Identificador de subred**: Escriba el identificador de la subred que usó durante la implementación de la plantilla de ASCS/SCS o el identificador de la subred que se creó como parte de la implementación de esta plantilla.

### <a name="application-servers-template"></a><a name="application-servers-template"></a>Plantilla de servidores de aplicaciones

La plantilla de servidores de aplicaciones permite implementar dos o más máquinas virtuales que se pueden usar como instancias de servidores de aplicaciones SAP en un sistema SAP. Por ejemplo, si ha implementado una plantilla de ASCS/SCS para 5 sistemas SAP, debe implementar esta plantilla cinco veces.

Para configurar la plantilla de varios SID de servidores de aplicaciones, en la [plantilla de varios SID de servidores de aplicaciones][sap-templates-3-tier-multisid-apps-marketplace-image] o en la [plantilla de varios SID de servidores de aplicaciones con Managed Disks][sap-templates-3-tier-multisid-apps-marketplace-image-md], especifique los valores para los parámetros siguientes:

  -  **Identificador de sistema SAP**: escriba el identificador del sistema SAP que se va a instalar. El identificador se usa como prefijo de los recursos que se implementan.
  -  **Tipo de SO**: Seleccione el sistema operativo de las máquinas virtuales.
  -  **Tamaño del sistema SAP**: La cantidad de SAPS que el sistema nuevo proporciona. Si no está seguro de cuántos SAPS necesita el sistema, consulte con el integrador de sistemas o el asociado tecnológico de SAP.
  -  **Disponibilidad del sistema**: Seleccione **Alta disponibilidad**.
  -  **Nombre de usuario y contraseña del administrador**: Cree un nuevo usuario que pueda usarse para iniciar sesión en la máquina.
  -  **Identificador de subred**: Escriba el identificador de la subred que usó durante la implementación de la plantilla de ASCS/SCS o el identificador de la subred que se creó como parte de la implementación de esta plantilla.


## <a name="azure-virtual-network"></a><a name="47d5300a-a830-41d4-83dd-1a0d1ffdbe6a"></a> Azure Virtual Network
En nuestro ejemplo, el espacio de direcciones de la instancia de Azure Virtual Network es 10.0.0.0/16. Hay una subred llamada Subnet, con un intervalo de direcciones de 10.0.0.0/24. Todas las máquinas virtuales y los equilibradores de carga internos están implementados en esta red virtual.

> [!IMPORTANT]
> No haga ningún cambio en la configuración de red dentro del sistema operativo invitado. Esto incluye direcciones IP, servidores DNS y subred. Ajuste toda la configuración de red de Azure. El servicio Protocolo de configuración dinámica de host (DHCP) propaga la configuración.
>
>

## <a name="dns-ip-addresses"></a><a name="b22d7b3b-4343-40ff-a319-097e13f62f9e"></a> Direcciones IP de DNS

Para configurar las direcciones IP de DNS que se requieren, realice los pasos siguientes:

1. En Azure Portal, en el panel **Servidores DNS**, asegúrese de que la opción **Servidores DNS** de la red virtual está establecida en **DNS personalizado**.
2. Seleccione la configuración según el tipo de red que tiene. Para obtener más información, consulte los siguientes recursos:
   * agregue las direcciones IP de los servidores DNS locales.  
   Puede extender los servidores DNS locales a las máquinas virtuales que se ejecutan en Azure. En ese escenario, puede agregar las direcciones IP de las máquinas virtuales de Azure en las que ejecuta el servidor DNS.
   * Para implementaciones de máquina virtual que están aisladas en Azure: implemente una máquina virtual adicional en la misma instancia de Virtual Network que actúa como servidor DNS. Agregue las direcciones IP de las máquinas virtuales de Azure que configuró para ejecutar el servicio DNS.

   ![Ilustración 2: Configuración de servidores DNS para Azure Virtual Network][sap-ha-guide-figure-3001]

   _**Ilustración 2:** Configuración de servidores DNS para Azure Virtual Network_

   > [!NOTE]
   > Si cambia las direcciones IP de los servidores DNS, debe reiniciar las máquinas virtuales de Azure para aplicar el cambio y propagar los nuevos servidores DNS.
   >
   >

En este ejemplo, el servicio DNS está instalado y configurado en estas máquinas virtuales de Windows:

| Rol de máquina virtual | Nombre de host de máquina virtual | Nombre de tarjeta de red | Dirección IP estática |
| --- | --- | --- | --- |
| Primer servidor DNS |domcontr-0 |pr1-nic-domcontr-0 |10.0.0.10 |
| Segundo servidor DNS |domcontr-1 |pr1-nic-domcontr-1 |10.0.0.11 |

## <a name="host-names-and-static-ip-addresses-for-the-sap-ascsscs-clustered-instance-and-dbms-clustered-instance"></a><a name="9fbd43c0-5850-4965-9726-2a921d85d73f"></a> Nombres de host y direcciones IP estáticas para la instancia en clúster de ASCS/SCS de SAP y la instancia de clúster de DBMS

Para la implementación local, necesita estas direcciones IP y nombres de host reservados:

| Rol de nombre de host virtual | Nombre de host virtual | Dirección IP estática virtual |
| --- | --- | --- |
| Primer nombre de host virtual de clúster de ASCS/SCS de SAP (para la administración del clúster) |pr1-ascs-vir |10.0.0.42 |
| Nombre de host virtual de la instancia de ASCS/SCS de SAP |pr1-ascs-sap |10.0.0.43 |
| Segundo nombre de host virtual de clúster de DBMS de SAP (administración del clúster) |pr1-dbms-vir |10.0.0.32 |

Cuando crea el clúster, crea los nombres de host virtual pr1-ascs-vir y pr1-dbms-vir, además de las direcciones IP asociadas que administran el clúster mismo. Para más información sobre cómo hacerlo, consulte [Recopilación de nodos del clúster en la configuración de clúster][sap-high-availability-infrastructure-wsfc-shared-disk-collect-cluster-config].

Puede crear manualmente los otros dos nombres de host virtual, pr1-ascs-sap y pr1-dbms-sap y las direcciones IP asociadas en el servidor DNS. La instancia de ASCS/SCS de SAP en clúster y la instancia de DBMS en clúster usan estos recursos. Para más información sobre cómo hacerlo, vea [Creación de un nombre de host virtual para la instancia de ASCS/SCS de SAP en clúster][sap-ha-guide-9.1.1].

## <a name="set-static-ip-addresses-for-the-sap-virtual-machines"></a><a name="84c019fe-8c58-4dac-9e54-173efd4b2c30"></a> Configuración de direcciones IP estáticas para las máquinas virtuales SAP
Después de implementar las máquinas virtuales que usará en el clúster, deberá establecer direcciones IP estáticas para todas las máquinas virtuales. Debe hacerlo en la configuración de Azure Virtual Network y no en el sistema operativo invitado.

1. En Azure Portal, seleccione **Grupo de recursos** > **Tarjeta de red** > **Configuración** > **Dirección IP**.
2. En el panel **Direcciones IP**, en **Asignación**, seleccione **Estática**. En el cuadro **Dirección IP**, escriba la dirección IP que desea usar.

   > [!NOTE]
   > Si cambia la dirección IP de la tarjeta de red, deberá reiniciar las máquinas virtuales de Azure para aplicar el cambio.  
   >
   >

   ![Ilustración 3: Establecimiento de direcciones IP estáticas para la tarjeta de red de cada máquina virtual][sap-ha-guide-figure-3002]

   _**Ilustración 3:** Establecimiento de direcciones IP estáticas para la tarjeta de red de cada máquina virtual_

   Repita este paso para todas las interfaces de red, es decir, para todas las máquinas virtuales, incluidas las que desea usar para Active Directory o el servicio DNS.

En este ejemplo, aparecen estas máquinas virtuales y direcciones IP estáticas:

| Rol de máquina virtual | Nombre de host de máquina virtual | Nombre de tarjeta de red | Dirección IP estática |
| --- | --- | --- | --- |
| Primer servidor de aplicaciones de SAP |pr1-di-0 |pr1-nic-di-0 |10.0.0.50 |
| Segundo servidor de aplicaciones de SAP |pr1-di-1 |pr1-nic-di-1 |10.0.0.51 |
| … |… |… |… |
| Último servidor de aplicaciones de SAP |pr1-di-5 |pr1-nic-di-5 |10.0.0.55 |
| Primer nodo de clúster para la instancia de ASCS/SCS |pr1-ascs-0 |pr1-nic-ascs-0 |10.0.0.40 |
| Segundo nodo de clúster para la instancia de ASCS/SCS |pr1-ascs-1 |pr1-nic-ascs-1 |10.0.0.41 |
| Primer nodo de clúster para la instancia de DBMS |pr1-db-0 |pr1-nic-db-0 |10.0.0.30 |
| Segundo nodo de clúster para la instancia de DBMS |pr1-db-1 |pr1-nic-db-1 |10.0.0.31 |

## <a name="set-a-static-ip-address-for-the-azure-internal-load-balancer"></a><a name="7a8f3e9b-0624-4051-9e41-b73fff816a9e"></a> Configuración de una dirección IP estática para el equilibrador de carga interno de Azure

La plantilla de Azure Resource Manager para SAP crea un equilibrador de carga interno de Azure que se usa para el clúster de instancia de ASCS/SCS de SAP y el clúster de DBMS.

> [!IMPORTANT]
> La dirección IP del nombre de host virtual de la instancia de ASCS/SCS de SAP es la misma que la dirección IP del equilibrador de carga interno de la instancia de ASCS/SCS de SAP, pr1-lb-ascs.
> La dirección IP del nombre virtual de la instancia de DBMS es la misma que la dirección IP del equilibrador de carga interno de DBMS, pr1-lb-dbms.
>
>

Para configurar una dirección IP estática para el equilibrador de carga interno de Azure, siga estos pasos:

1. En la implementación inicial se establece la dirección IP del equilibrador de carga interno como **Dinámica**. En el panel **Direcciones IP** de Azure Portal, en **Asignación**, seleccione **Estática**.
2. Establezca la dirección IP del equilibrador de carga interno **pr1-lb-ascs** en la dirección IP del nombre de host virtual de la instancia de ASCS/SCS de SAP.
3. Establezca la dirección IP del equilibrador de carga interno **pr1-lb-dbms** en la dirección IP del nombre de host virtual de la instancia de DBMS.

   ![Ilustración 4: Establecimiento de direcciones IP estáticas para el equilibrador de carga interno de la instancia de ASCS/SCS de SAP][sap-ha-guide-figure-3003]

   _**Ilustración 4:** Establecimiento de direcciones IP estáticas para el equilibrador de carga interno de la instancia de ASCS/SCS de SAP_

En este ejemplo, aparecen 2 equilibradores de carga internos de Azure que tienen estas direcciones IP estáticas:

| Rol del equilibrador de carga interno de Azure | Nombre del equilibrador de carga interno de Azure | Dirección IP estática |
| --- | --- | --- |
| Equilibrador de carga interno de instancia de ASCS/SCS de SAP |pr1-lb-ascs |10.0.0.43 |
| Equilibrador de carga interno de DBMS de SAP |pr1-lb-dbms |10.0.0.33 |


## <a name="default-ascsscs-load-balancing-rules-for-the-azure-internal-load-balancer"></a><a name="f19bd997-154d-4583-a46e-7f5a69d0153c"></a> Reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure

La plantilla de Azure Resource Manager para SAP crea los puertos que necesita:
* Una instancia de ASCS de ABAP con el número de instancia predeterminado 00
* Una instancia de SCS de Java con el número de instancia predeterminado 01

Cuando instala la instancia de ASCS/SCS de SAP, debe usar el número de instancia predeterminado 00 para la instancia de ASCS ABAP y el número de instancia predeterminado 01 para la instancia de SCS de Java.

Después, cree los puntos de conexión del equilibrio de carga interno necesarios para los puertos de SAP NetWeaver.

Para crear estos puntos de conexión de equilibrio de carga interno necesarios, créelos para los puertos de ASCS ABAP de SAP NetWeaver:

| Nombre de regla de equilibrio de carga/servicio | Números de puerto predeterminados | Puertos concretos para (instancia de ASCS con el número de instancia 00) (ERS con 10) |
| --- | --- | --- |
| Servidor de colas / *lbrule3200* |32\<InstanceNumber\> |3200 |
| Servidor de mensajes de ABAP / *lbrule3600* |36\<InstanceNumber\> |3600 |
| Mensaje de ABAP interno / *lbrule3900* |39\<InstanceNumber\> |3900 |
| Servidor de mensajes HTTP / *Lbrule8100* |81\<InstanceNumber\> |8100 |
| Servicio de inicio de SAP ASCS HTTP/ *Lbrule50013* |5\<InstanceNumber\>13 |50013 |
| Servicio de inicio de SAP ASCS HTTP / *Lbrule50014* |5\<InstanceNumber\>14 |50014 |
| Replicación de colas / *Lbrule50016* |5\<InstanceNumber\>16 |50016 |
| Servicio de inicio de SAP ERS HTTP/ *Lbrule51013* |5\<InstanceNumber\>13 |51013 |
| Servicio de inicio de SAP ERS HTTP/ *Lbrule51014* |5\<InstanceNumber\>14 |51014 |
| Administración remota de Windows (WinRM) *Lbrule5985* | |5985 |
| Recurso compartido de archivos *Lbrule445* | |445 |

**Tabla 1:** números de puerto de las instancias de ASCS de ABAP de SAP NetWeaver.

Luego, cree estos puntos de conexión de equilibrio de carga interno para los puertos de SCS de Java de SAP NetWeaver:

| Nombre de regla de equilibrio de carga/servicio | Números de puerto predeterminados | Puertos concretos para (instancia de SCS con el número de instancia 01) (ERS con 11) |
| --- | --- | --- |
| Servidor de colas / *lbrule3201* |32\<InstanceNumber\> |3201 |
| Servidor de puerta de enlace / *lbrule3301* |33\<InstanceNumber\> |3301 |
| Servidor de mensajes de Java / *lbrule3900* |39\<InstanceNumber\> |3901 |
| Servidor de mensajes HTTP/ *Lbrule8101* |81\<InstanceNumber\> |8101 |
| Servicio de inicio de SAP SCS HTTP/ *Lbrule50113* |5\<InstanceNumber\>13 |50113 |
| Servicio de inicio de SAP SCS HTTPS/ *Lbrule50114* |5\<InstanceNumber\>14 |50114 |
| Replicación de colas / *Lbrule50116* |5\<InstanceNumber\>16 |50116 |
| Servicio de inicio de SAP ERS HTTP *Lbrule51113* |5\<InstanceNumber\>13 |51113 |
| Servicio de inicio de SAP ERS HTTPS/ *Lbrule51114* |5\<InstanceNumber\>14 |51114 |
| WinRM/ *Lbrule5985* | |5985 |
| Recurso compartido de archivos *Lbrule445* | |445 |

**Tabla 2:** números de puerto de las instancias de SCS de Java de SAP NetWeaver.

![Ilustración 5: Reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure][sap-ha-guide-figure-3004]

_**Ilustración 5:** Reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure_

Establezca la dirección IP del equilibrador de carga pr1-lb-dbms en la dirección IP del nombre de host virtual de la instancia de DBMS.

### <a name="change-the-ascsscs-default-load-balancing-rules-for-the-azure-internal-load-balancer"></a><a name="fe0bd8b5-2b43-45e3-8295-80bee5415716"></a> Cambio de las reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure

Si desea usar otros números para las instancias de ASCS o SCS de SAP, debe actualizar los nombres y valores de sus puertos a partir de los predeterminados.

1. En Azure Portal, seleccione **\<SID\>equilibrador de carga -lb-ascs** > **Reglas de equilibrio de carga**.
2. Cambie estos valores para todas las reglas de equilibrio de carga que pertenezcan a la instancia de ASCS o SCS de SAP:

   * Nombre
   * Port
   * Puerto de back-end

   Por ejemplo, si desea cambiar el número de instancia de ASCS predeterminado de 00 a 31, debe hacer cambios en todos los puertos que aparecen en la tabla 1.

   A continuación, se muestra un ejemplo de una actualización para el puerto *lbrule3200*.

   ![Figura 6: Cambio de las reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure][sap-ha-guide-figure-3005]

   _**Figura 6:** Cambio de las reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure_

## <a name="add-windows-virtual-machines-to-the-domain"></a><a name="e69e9a34-4601-47a3-a41c-d2e11c626c0c"></a> Incorporación de máquinas virtuales de Windows al dominio

Después de asignar una dirección IP estática a las máquinas virtuales, agregue las máquinas virtuales al dominio.

![Figura 7: Incorporación de una máquina virtual a un dominio][sap-ha-guide-figure-3006]

_**Figura 7:** Incorporación de una máquina virtual a un dominio_

## <a name="add-registry-entries-on-both-cluster-nodes-of-the-sap-ascsscs-instance"></a><a name="661035b2-4d0f-4d31-86f8-dc0a50d78158"></a> Incorporación de entradas del Registro en ambos nodos del clúster usados para la instancia de ASCS/SCS de SAP

Azure Load Balancer tiene un equilibrador de carga interno que cierra las conexiones cuando están inactivas durante un período establecido (tiempo de espera de inactividad). Los proceso de trabajo de SAP en instancias de diálogo abren conexiones con el proceso de puesta en cola de SAP tan pronto como se deba enviar la primera solicitud para poner en cola y para quitar de la cola. Estas conexiones suelen mantenerse hasta que el proceso de trabajo o el de puesta en cola se reinicia. Sin embargo, si la conexión está inactiva durante un periodo, el equilibrador de carga interno de Azure cierra la conexión. Esto no supone un problema, porque el proceso de trabajo de SAP restablece la conexión con el proceso de puesta en cola si ya no existe. Estas actividades se documentan en los seguimientos de desarrollador de los procesos de SAP, pero crean una gran cantidad de contenido adicional en esos seguimientos. Se recomienda cambiar los parámetros de TCP/IP `KeepAliveTime` y `KeepAliveInterval` en ambos nodos del clúster. Combine estos cambios en los parámetros de TCP/IP con los parámetros de perfil de SAP, lo que se describe más adelante en este artículo.

Para agregar entradas de registro en los dos nodos de clúster de la instancia de ASCS/SCS de SAP, en primer lugar, agregue estas entradas del registro de Windows en ambos nodos de clúster de Windows para ASCS/SCS de SAP:

| Path | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| Nombre de la variable |`KeepAliveTime` |
| Tipo de variable |REG_DWORD (Decimal) |
| Value |120000 |
| Vínculo a la documentación |[https://technet.microsoft.com/library/cc957549.aspx](/previous-versions/windows/it-pro/windows-2000-server/cc957549(v=technet.10)) |

**Tabla 3:** Cambio del primer parámetro de TCP/IP

Luego, agregue estas entradas del registro de Windows en ambos nodos del clúster de Windows para ASCS/SCS de SAP:

| Path | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| Nombre de la variable |`KeepAliveInterval` |
| Tipo de variable |REG_DWORD (Decimal) |
| Value |120000 |
| Vínculo a la documentación |[https://technet.microsoft.com/library/cc957548.aspx](/previous-versions/windows/it-pro/windows-2000-server/cc957548(v=technet.10)) |

**Tabla 4:** Cambio del segundo parámetro de TCP/IP

Para aplicar los cambios, reinicie ambos nodos del clúster.

## <a name="set-up-a-windows-server-failover-cluster-for-an-sap-ascsscs-instance"></a><a name="0d67f090-7928-43e0-8772-5ccbf8f59aab"></a> Configuración de un clúster de conmutación por error de Windows Server para una instancia de ASCS/SCS de SAP

Para configurar un clúster de conmutación por error de Windows Server para una instancia de ASCS/SCS de SAP, hay que realizar estas tareas:

- Recopilación de los nodos del clúster en una configuración de clúster.
- Configuración de un testigo de recurso compartido de archivos de clúster.

### <a name="collect-the-cluster-nodes-in-a-cluster-configuration"></a><a name="5eecb071-c703-4ccc-ba6d-fe9c6ded9d79"></a> Recopilación de nodos del clúster en la configuración de clúster

1. En el Asistente para Agregar roles y características, agregue a ambos nodos del clúster agrupaciones en clústeres de conmutación por error.
2. Configure el clúster de conmutación por error mediante el Administrador de clústeres de conmutación por error. En Administrador de clústeres de conmutación por error, seleccione **Crear clúster** y, luego, agregue solo el nombre del primer clúster (nodo A). No agregue el segundo nodo aún; el segundo nodo se agrega en un paso posterior.

   ![Figura 8: Incorporación del nombre de servidor o máquina virtual del primer nodo del clúster][sap-ha-guide-figure-3007]

   _**Figura 8:** Incorporación del nombre de servidor o máquina virtual del primer nodo del clúster_

3. Escriba el nombre de red (nombre de host virtual) del clúster.

   ![Figura 9: Escriba el nombre del clúster][sap-ha-guide-figure-3008]

   _**Figura 9:** Escriba el nombre del clúster_

4. Una vez creado el clúster, ejecute una prueba de validación del clúster.

   ![Figura 10: Ejecución de la comprobación de validación del clúster][sap-ha-guide-figure-3009]

   _**Figura 10:** Ejecución de la comprobación de validación del clúster_

   En este punto del proceso, puede omitir cualquier advertencia sobre los discos. Agregará un testigo de recurso de archivos y los discos compartidos de SIOS más tarde. En esta fase, no es necesario preocuparse por el cuórum.

   ![Figura 11: No se encuentra disco de cuórum][sap-ha-guide-figure-3010]

   _**Figura 11:** No se encuentra disco de cuórum_

   ![Figura 12: Un recurso de clúster principal necesita una dirección IP nueva][sap-ha-guide-figure-3011]

   _**Figura 12:** Un recurso de clúster principal necesita una dirección IP nueva_

5. Cambie la dirección IP del servicio de clúster principal. El clúster no se puede iniciar hasta que no cambie la dirección IP del servicio de clúster principal, porque la dirección IP del servidor apunta a uno de los nodos de la máquina virtual. Para ello, vaya a la página **Propiedad** del recurso de dirección IP del servicio de clúster principal.

   Por ejemplo, necesitamos asignar una dirección IP (10.0.0.42 en el ejemplo) para el nombre de host virtual del clúster pr1-ascs-vir.

   ![Figura 13: En el cuadro de diálogo Propiedades, cambie la dirección IP][sap-ha-guide-figure-3012]

   _**Figura 13:** En el cuadro de diálogo **Propiedades**, cambie la dirección IP_

   ![Figura 14: Asignación de la dirección IP reservada para el clúster][sap-ha-guide-figure-3013]

   _**Figura 14:** Asignación de la dirección IP reservada para el clúster_

6. Escriba el nombre del host virtual del clúster en línea.

   ![Figura 15: El servicio principal del clúster está activo, en ejecución y tiene la dirección IP correcta][sap-ha-guide-figure-3014]

   _**Figura 15:** El servicio principal del clúster está activo, en ejecución y tiene la dirección IP correcta_

7. Agregue el segundo nodo del clúster.

   Ahora que el servicio de clúster principal está en funcionamiento, puede agregar el segundo nodo del clúster.

   ![Figura 16: Incorporación del segundo nodo del clúster][sap-ha-guide-figure-3015]

   _**Figura 16:** Incorporación del segundo nodo del clúster_

8. Escriba un nombre para el segundo host del nodo de clúster.

   ![Figura 17: Escriba el segundo nombre de host del nodo de clúster][sap-ha-guide-figure-3016]

   _**Figura 17:** Escriba el segundo nombre de host del nodo de clúster_

   > [!IMPORTANT]
   > Asegúrese de que la casilla **Agregar todo el almacenamiento apto al clúster** *no* esté activada.  
   >
   >

   ![Figura 18: No active la casilla][sap-ha-guide-figure-3017]

   _**Figura 18:** *No* active la casilla_

   Puede pasar por alto las advertencias sobre el cuórum y los discos. Se establece el cuórum y se comparte el disco más adelante, tal como se explica en [Instalación de SIOS DataKeeper Cluster Edition para un disco compartido de clúster de ASCS/SCS de SAP][sap-high-availability-infrastructure-wsfc-shared-disk-install-sios].

   ![Figura 19: Omisión de las advertencias sobre el cuórum de disco][sap-ha-guide-figure-3018]

   _**Figura 19:** Omisión de las advertencias sobre el cuórum de disco_


#### <a name="configure-a-cluster-file-share-witness"></a><a name="e49a4529-50c9-4dcf-bde7-15a0c21d21ca"></a> Configuración de un testigo de recurso compartido de archivos de clúster

Para configurar un testigo de recurso compartido de archivos de clúster, hay que realizar las siguientes tareas:

- Creación de un recurso compartido de archivos.
- Configuración del cuórum del testigo del recurso compartido de archivos en el Administrador de clústeres de conmutación por error.

#### <a name="create-a-file-share"></a><a name="06260b30-d697-4c4d-b1c9-d22c0bd64855"></a> Creación de un recurso compartido de archivos

1. Seleccione un testigo de recurso de archivos en lugar de un disco de cuórum. SIOS DataKeeper admite esta opción.

   En los ejemplos de este artículo, el testigo del recurso compartido de archivos se encuentra en la instancia de Active Directory o el servidor DNS que se ejecuta en Azure. El testigo del recurso compartido de archivos se llama domcontr-0. Dado que habría configurado una conexión de VPN con Azure (a través de VPN Gateway o Azure ExpressRoute), su instancia de Active Directory o del servidor DNS es local y no es apta para ejecutar un testigo del recurso compartido de archivos.

   > [!NOTE]
   > Si la instancia de Active Directory o el servidor DNS solo se ejecutan localmente, no configure el testigo del recurso compartido de archivos en la instancia de Active Directory o sistema operativo Windows de DNS que se ejecuta localmente. La latencia de red entre los nodos del clúster que se ejecutan en Azure y Active Directory o DNS local puede ser demasiado grande y provocar problemas de conectividad. Asegúrese de configurar el testigo de recurso compartido de archivos en una máquina virtual de Azure que se ejecuta cerca del nodo del clúster.  
   >
   >

   La unidad de cuórum necesita, como mínimo, 1024 MB de espacio disponible. Se recomienda 2048 MB de espacio disponible en la unidad de cuórum.

2. Agregue el objeto de nombre de clúster.

   ![Figura 20: Asignación de los permisos en el recurso compartido para el objeto de nombre de clúster][sap-ha-guide-figure-3019]

   _**Figura 20:** Asignación de los permisos en el recurso compartido para el objeto de nombre de clúster_

   Asegúrese de que los permisos incluyan la autoridad para cambiar datos en el recurso compartido para el objeto de nombre de clúster (pr1-ascs-vir$ en el ejemplo).

3. Para agregar el objeto de nombre de clúster a la lista, seleccione **Agregar**. Cambie el filtro para buscar objetos de equipo, además de los que aparecen en la figura 22.

   ![Figura 21: Cambio de los tipos de objeto para incluir los equipos][sap-ha-guide-figure-3020]

   _**Figura 21:** Cambio de los **tipos de objeto** para incluir los equipos_

   ![Figura 22: Active la casilla Equipos][sap-ha-guide-figure-3021]

   _**Figura 22:** Active la casilla **Equipos**_

4. Escriba el objeto de nombre de clúster, como aparece en la figura 21. Como ya se creó el registro, puede cambiar los permisos, tal como aparece en la figura 20.

5. Seleccione la pestaña **Seguridad** del recurso compartido y, luego, establezca permisos más detallados para el objeto de nombre de clúster.

   ![Figura 23: Establecimiento de los atributos de seguridad para el objeto de nombre de clúster en el cuórum de recurso compartido de archivos][sap-ha-guide-figure-3022]

   _**Figura 23:** Establecimiento de los atributos de seguridad para el objeto de nombre de clúster en el cuórum de recurso compartido de archivos_

#### <a name="set-the-file-share-witness-quorum-in-failover-cluster-manager"></a><a name="4c08c387-78a0-46b1-9d27-b497b08cac3d"></a> Configuración del cuórum de testigo del recurso compartido de archivos en el Administrador de clústeres de conmutación por error

1. Abra el Asistente para la configuración de cuórum.

   ![Figura 24: Inicio del Asistente para configurar el cuórum de clúster][sap-ha-guide-figure-3023]

   _**Figura 24:** Inicio del Asistente para configurar el cuórum de clúster_

2. En la página **Seleccionar opción de configuración de cuórum**, seleccione **Seleccionar el testigo de cuórum**.

   ![Figura 25: Configuraciones de cuórum que puede elegir][sap-ha-guide-figure-3024]

   _**Figura 25:** Configuraciones de cuórum que puede elegir_

3. En la página **Seleccionar testigo de cuórum**, elija **Configurar un testigo de recurso compartido de archivos**.

   ![Figura 26: Selección del testigo de recurso compartido de archivos][sap-ha-guide-figure-3025]

   _**Figura 26:** Selección del testigo de recurso compartido de archivos_

4. Escriba la ruta de acceso UNC al recurso compartido de archivos (\\domcontr-0\FSW en el ejemplo). Seleccione **Siguiente** para ver una lista de los cambios que puede hacer.

   ![Figura 27: Definición de la ubicación del recurso compartido de archivos para el recurso compartido testigo][sap-ha-guide-figure-3026]

   _**Figura 27:** Definición de la ubicación del recurso compartido de archivos para el recurso compartido testigo_

5. Seleccione los cambios que desee y, luego, seleccione **Siguiente**. Necesita volver a configurar correctamente el clúster, tal como aparece en la figura 28:  

   ![Figura 28: Confirmación de que volvió a configurar el clúster][sap-ha-guide-figure-3027]

   _**Figura 28:** Confirmación de que volvió a configurar el clúster_

Después de instalar correctamente el clúster de conmutación por error de Windows, se deben realizar cambios en algunos umbrales para adaptar la detección de conmutación por error a las condiciones de Azure. Los parámetros que se van a cambiar se documentan en [Ajuste de los umbrales de la red en clúster de conmutación por error][tuning-failover-cluster-network-thresholds]. Suponiendo que las dos máquinas virtuales que forman la configuración del clúster de Windows para ASCS/SCS están en la misma subred, los parámetros siguientes deben cambiarse a estos valores:

- SameSubNetDelay = 2000
- SameSubNetThreshold = 15
- RoutingHistoryLength = 30

Esta configuración se han probado con clientes y ofrece un buen compromiso. Son lo suficientemente resistentes y también proporcionan una conmutación por error que es lo suficientemente rápida en condiciones de error real en un software de SAP o en un nodo o un error de la máquina virtual.

### <a name="install-sios-datakeeper-cluster-edition-for-the-sap-ascsscs-cluster-share-disk"></a><a name="5c8e5482-841e-45e1-a89d-a05c0907c868"></a> Instalación de SIOS DataKeeper Cluster Edition para un disco compartido de clúster de ASCS/SCS de SAP

Ahora tiene una configuración de clústeres de conmutación por error de Windows Server activa en Azure. Para instalar una instancia de ASCS/SCS de SAP, necesita un recurso de disco compartido. No puede crear los recursos de disco compartido que necesita en Azure. SIOS DataKeeper Cluster Edition es una solución de terceros que puede usar para crear recursos de disco compartido.

Para instalar SIOS DataKeeper Cluster Edition en un disco compartido de clúster de ASCS/SCS de SAP, siga estos pasos:

- Adición de Microsoft .NET Framework 3.5.
- Instalación de SIOS DataKeeper.
- Configuración de SIOS DataKeeper.

### <a name="add-net-framework-35"></a><a name="1c2788c3-3648-4e82-9e0d-e058e475e2a3"></a> Incorporación de la característica .NET Framework 3.5
.NET Framework 3.5 no está activo ni instalado automáticamente en Windows Server 2012 R2. Dado que SIOS DataKeeper requiere que .NET esté instalado en todos los nodos donde se instala DataKeeper, debe instalar .NET Framework 3.5 en el sistema operativo invitado de todas las máquinas virtuales del clúster.

Hay dos maneras de agregar .NET Framework 3.5:

- Use el Asistente para agregar roles y características en Windows, como se muestra en la figura 29:

  ![Figura 29: Instalación de .NET Framework 3.5 por medio del Asistente para agregar roles y características][sap-ha-guide-figure-3028]

  _**Figura 29:** Instalación de .NET Framework 3.5 por medio del Asistente para agregar roles y características_

  ![Figura 30: Barra de progreso de la instalación de .NET Framework 3.5 por medio del Asistente para agregar roles y características][sap-ha-guide-figure-3029]

  _**Figura 30:** Barra de progreso de la instalación de .NET Framework 3.5 por medio del Asistente para agregar roles y características_

- Utilice la herramienta de línea de comandos dism.exe. Para este tipo de instalación, necesita tener acceso al directorio SxS en los medios de instalación de Windows. Escriba este comando en un símbolo del sistema con privilegios elevados:

  ```
  Dism /online /enable-feature /featurename:NetFx3 /All /Source:installation_media_drive:\sources\sxs /LimitAccess
  ```

### <a name="install-sios-datakeeper"></a><a name="dd41d5a2-8083-415b-9878-839652812102"></a> Instalación de SIOS DataKeeper

Instale SIOS DataKeeper Cluster Edition en cada nodo del clúster. Con SIOS DataKeeper, para crear el almacenamiento compartido virtual, cree un reflejo sincronizado y, luego, simule el almacenamiento compartido de clúster.

Antes de instalar el software de SIOS, cree el usuario de dominio DataKeeperSvc.

> [!NOTE]
> Agregue el usuario DataKeeperSvc al grupo de administradores locales en ambos nodos del clúster.
>
>

Para instalar SIOS DataKeeper, siga estos pasos:

1. Instale el software SIOS en ambos nodos del clúster.

   ![Instalador de SIOS][sap-ha-guide-figure-3030]

   ![Figura 31: Primera página de la instalación de SIOS DataKeeper][sap-ha-guide-figure-3031]

   _**Figura 31:** Primera página de la instalación de SIOS DataKeeper_

2. En el cuadro de diálogo, seleccione **Sí**.

   ![Figura 32: DataKeeper le informa que se deshabilitará un servicio][sap-ha-guide-figure-3032]

   _**Figura 32:** DataKeeper le informa que se deshabilitará un servicio_

3. En el cuadro de diálogo, se recomienda seleccionar **Cuenta de dominio o de servidor**.

   ![Figura 33: Selección del usuario para SIOS DataKeeper][sap-ha-guide-figure-3033]

   _**Figura 33:** Selección del usuario para SIOS DataKeeper_

4. Escriba el nombre de usuario y la contraseña de la cuenta de dominio que creó para SIOS DataKeeper.

   ![Figura 34: Introducción del nombre de usuario y contraseña de dominio para la instalación de SIOS DataKeeper][sap-ha-guide-figure-3034]

   _**Figura 34:** Introducción del nombre de usuario y contraseña de dominio para la instalación de SIOS DataKeeper_

5. Instale la clave de licencia para la instancia de SIOS DataKeeper, tal como aparece en la figura 35.

   ![Figura 35: Especificación de la licencia de SIOS DataKeeper][sap-ha-guide-figure-3035]

   _**Figura 35:** Especificación de la licencia de SIOS DataKeeper_

6. Cuando se le solicite, reinicie la máquina virtual.

### <a name="set-up-sios-datakeeper"></a><a name="d9c1fc8e-8710-4dff-bec2-1f535db7b006"></a> Configuración de SIOS DataKeeper

Después de instalar SIOS DataKeeper en ambos nodos, inicie la configuración. El objetivo de la configuración es conseguir la replicación sincrónica de datos entre los discos adicionales conectados a cada una de las máquinas virtuales.

1. Inicie la herramienta de configuración y administración de DataKeeper y, luego, seleccione **Servidor de conexión**.

   ![Figura 36: Herramienta de configuración y administración de SIOS DataKeeper][sap-ha-guide-figure-3036]

   _**Figura 36:** Herramienta de configuración y administración de SIOS DataKeeper_

2. Escriba el nombre o la dirección TCP/IP del primer nodo al que se debe conectar la herramienta de configuración y administración y, en un segundo paso, el segundo nodo.

   ![Figura 37: Inserte el nombre o la dirección TCP/IP del primer nodo al que se debe conectar la herramienta de configuración y administración y, en un segundo paso, el segundo nodo][sap-ha-guide-figure-3037]

   _**Figura 37:** Inserte el nombre o la dirección TCP/IP del primer nodo al que se debe conectar la herramienta de configuración y administración y, en un segundo paso, el segundo nodo_

3. Cree el trabajo de replicación entre los dos nodos.

   ![Figura 38: Creación de un trabajo de replicación][sap-ha-guide-figure-3038]

   _**Figura 38:** Creación de un trabajo de replicación_

   Un asistente le guía por el proceso de crear un trabajo de replicación.

4. Defina el nombre del trabajo de replicación.

   ![Figura 39: Definición del nombre del trabajo de replicación][sap-ha-guide-figure-3039]

   _**Figura 39:** Definición del nombre del trabajo de replicación_

   ![Figura 40: Definición de los datos básicos para el nodo que debe ser el nodo de origen actual][sap-ha-guide-figure-3040]

   _**Figura 40:** Definición de los datos básicos para el nodo que debe ser el nodo de origen actual_

5. Defina el nombre, la dirección TCP/IP y el volumen de disco del nodo de destino.

   ![Figura 41: Definición del nombre, la dirección TCP/IP y el volumen de disco del nodo de destino actual][sap-ha-guide-figure-3041]

   _**Figura 41:** Definición del nombre, la dirección TCP/IP y el volumen de disco del nodo de destino actual_

6. Defina los algoritmos de compresión. En el ejemplo, se recomienda comprimir el flujo de replicación. Especialmente en situaciones de resincronización, la compresión del flujo de replicación reduce considerablemente el tiempo que se tarda en resincronizar. La compresión usa los recursos de CPU y RAM de una máquina virtual. A medida que aumenta la tasa de compresión, también aumenta el volumen de los recursos de CPU usados. Puede ajustar esta configuración más adelante.

7. Otra configuración que debe comprobar es si la replicación se ejecuta de forma sincrónica o asincrónica. Cuando proteja configuraciones de ASCS/SCS de SAP, debe usar la replicación sincrónica.  

   ![Figura 42: Definición de los detalles de la replicación][sap-ha-guide-figure-3042]

   _**Figura 42:** Definición de los detalles de la replicación_

8. Defina si el volumen que el trabajo de replicación replica se debe representar en una configuración de clúster de conmutación por error de Windows Server como disco compartido. Para la configuración de ASCS/SCS de SAP, seleccione **Sí** de manera que el clúster de Windows vea el volumen replicado como un disco compartido que puede usar como volumen de clúster.

   ![Figura 43: Selección de Sí para establecer el volumen replicado como volumen de clúster][sap-ha-guide-figure-3043]

   _**Figura 43:** Selección de **Sí** para establecer el volumen replicado como volumen de clúster_

   Una vez creado el volumen, la herramienta de configuración y administración de DataKeeper muestra que el trabajo de replicación está activo.

   ![Figura 44: El reflejo sincrónico de DataKeeper para el disco compartido de ASCS/SCS de SAP está activo][sap-ha-guide-figure-3044]

   _**Figura 44:** El reflejo sincrónico de DataKeeper para el disco compartido de ASCS/SCS de SAP está activo_

   Ahora, el Administrador de clústeres de conmutación por error muestra el disco como un disco de DataKeeper, tal como aparece en la figura 45:

   ![Figura 45: El Administrador de clústeres de conmutación por error muestra el disco que DataKeeper replicó][sap-ha-guide-figure-3045]

   _**Figura 45:** El Administrador de clústeres de conmutación por error muestra el disco que DataKeeper replicó_

## <a name="next-steps"></a>Pasos siguientes

* [Instalación de alta disponibilidad para SAP NetWeaver con un clúster de conmutación por error de Windows y un disco compartido para una instancia de ASCS/SCS de SAP][sap-high-availability-installation-wsfc-shared-disk]
