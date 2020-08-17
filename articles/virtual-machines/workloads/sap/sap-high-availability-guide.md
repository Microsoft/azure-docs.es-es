---
title: Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver
description: Guía de alta disponibilidad para SAP NetWeaver en Azure Virtual Machines
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
ms.date: 05/05/2017
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: dbdbae3d310d6e4c3224663dd523cb124744dfbd
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080189"
---
# <a name="high-availability-azure-virtual-machines-for-sap-netweaver"></a>Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[sap-installation-guides]:http://service.sap.com/instguides

[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md

[dbms-guide]:dbms-guide.md

[deployment-guide]:deployment-guide.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md

[ha-guide]:sap-high-availability-guide.md

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



Azure Virtual Machines es la solución para organizaciones que necesitan recursos de red, almacenamiento y proceso, en un tiempo mínimo y sin ciclos de adquisición prolongados. Puede usar Azure Virtual Machines para implementar aplicaciones clásicas como ABAP basado en SAP NetWeaver, Java y la pila de ABAP+Java. Amplíe la confiabilidad y disponibilidad sin recursos locales adicionales. Azure Virtual Machines admite la conectividad entre locales, así que podrá integrar Azure Virtual Machines a los dominios locales, las nubes privadas y la infraestructura de sistema SAP de la organización.

En este artículo analizaremos los pasos que puede llevar a cabo para implementar sistemas SAP de alta disponibilidad en Azure con el uso del modelo de implementación de Azure Resource Manager. Le guiaremos por estas tareas principales:

* Encontrar las notas y guías de instalación de SAP adecuadas, que aparecen en la sección [Recursos][sap-ha-guide-2]. Este documento complementa la documentación sobre la instalación de SAP y las notas de SAP, que son los recursos principales que lo ayudan a instalar e implementar software de SAP en plataformas específicas.
* Conocer las diferencias entre el modelo de implementación clásica de Azure y el modelo de implementación de Azure Resource Manager.
* Obtener información sobre los modos de cuórum de Clústeres de conmutación por error de Windows Server, a fin de poder seleccionar el modelo adecuado para la implementación de Azure.
* Obtener información sobre el almacenamiento compartido de Clústeres de conmutación por error de Windows Server en los servicios de Azure.
* Obtener información sobre cómo proteger en Azure los componentes con un único punto de error, como Advanced Business Application Programming (ABAP) SAP Central Services (ASCS)/SAP Central Services (SCS) y sistemas de administración de bases de datos (DBMS), además de componentes redundantes como los servidores de aplicaciones de SAP.
* Seguir un ejemplo detallado de la instalación y configuración de un sistema SAP de alta disponibilidad en un clúster de Clústeres de conmutación por error de Windows Server mediante Azure Resource Manager.
* Obtener información sobre los pasos adicionales que se necesitan para usar Clústeres de conmutación por error de Windows Server en Azure, pero que no son necesarios en una implementación local.

Para simplificar la implementación y configuración, en este artículo usamos las nuevas plantillas de Resource Manager de alta disponibilidad y de tres niveles de SAP. Las plantillas automatizan la implementación de toda la infraestructura que necesita en un sistema SAP de alta disponibilidad. La infraestructura también admite el ajuste de tamaño de SAP Application Performance Standard (SAPS) del sistema SAP.

## <a name="prerequisites"></a><a name="217c5479-5595-4cd8-870d-15ab00d4f84c"></a> Requisitos previos
Antes de comenzar, asegúrese de cumplir los requisitos previos que se describen en las secciones siguientes. Además, asegúrese de revisar todos los recursos que aparecen en la sección [Recursos][sap-ha-guide-2].

En este artículo usamos plantillas de Azure Resource Manager para [SAP NetWeaver de tres niveles con Managed Disks](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md/). Consulte [Plantillas de Azure Resource Manager para SAP](/archive/blogs/saponsqlserver/azure-quickstart-templates-for-sap) para ver información general útil sobre las plantillas.

## <a name="resources"></a><a name="42b8f600-7ba3-4606-b8a5-53c4f026da08"></a> Recursos
Estos artículos también se refieren a las implementaciones de SAP en Azure:

* [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver][planning-guide]
* [Implementación de Azure Virtual Machines para SAP NetWeaver][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP NetWeaver][dbms-guide]
* [Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver (esta guía)][sap-ha-guide]

> [!NOTE]
> Siempre que sea posible, aparece un vínculo a la guía de instalación de SAP a la que se hace referencia (consulte las [guías de instalación de SAP][sap-installation-guides]). Para consultar los requisitos previos e información sobre el proceso de instalación, se recomienda leer cuidadosamente las guías de instalación de SAP NetWeaver. En este artículo solo se abarcan tareas específicas para los sistemas basados en SAP NetWeaver que puede usar con Azure Virtual Machines.
>
>

Estas notas de SAP están relacionadas con el tema de SAP en Azure:

| Número de nota | Título |
| --- | --- |
| [1928533] |SAP Applications on Azure: Supported Products and Sizing (Aplicaciones de SAP en Microsoft Azure con Base de datos de Oracle: versiones y tamaños compatibles) |
| [2015553] |SAP on Microsoft Azure: Support Prerequisites (Requisitos previos de soporte técnico de SAP en Microsoft Azure) |
| [1999351] |Enhanced Azure Monitoring for SAP (Supervisión mejorada de Azure para SAP) |
| [2178632] |Key Monitoring Metrics for SAP on Microsoft Azure (Métricas de supervisión clave para SAP en Microsoft Azure) |
| [1999351] |Virtualization on Windows: Enhanced Monitoring (Virtualización en Windows: supervisión mejorada) |
| [2243692] |Uso de almacenamiento SSD premium de Azure para la instancia DBMS de SAP |

Obtenga más información sobre las [limitaciones de las suscripciones de Azure][azure-resource-manager/management/azure-subscription-service-limits-subscription], incluidas las limitaciones máximas y predeterminadas generales.

## <a name="high-availability-sap-with-azure-resource-manager-vs-the-azure-classic-deployment-model"></a><a name="42156640c6-01cf-45a9-b225-4baa678b24f1"></a>SAP de alta disponibilidad con el modelo de implementación de Azure Resource Manager en comparación con el modelo de implementación clásica
El modelo de implementación de Azure Resource Manager y el modelo de implementación clásica se diferencian de las siguientes maneras:

- Grupos de recursos
- Dependencia del equilibrador de carga interno de Azure en el grupo de recursos de Azure
- Soporte técnico para el escenario de varios SID de SAP

### <a name="resource-groups"></a><a name="f76af273-1993-4d83-b12d-65deeae23686"></a> Grupos de recursos
En Azure Resource Manager, puede usar los grupos de recursos para administrar todos los recursos de aplicación de la suscripción de Azure. En un enfoque integrado, todos los recursos de un grupo de recursos tienen el mismo ciclo de vida. Por ejemplo, todos los recursos se crean en el mismo momento y se eliminan de la misma manera. Más información sobre los [grupos de recursos](../../../azure-resource-manager/management/overview.md#resource-groups).

### <a name="azure-internal-load-balancer-dependency-on-the-azure-resource-group"></a><a name="3e85fbe0-84b1-4892-87af-d9b65ff91860"></a> Dependencia del equilibrador de carga interno de Azure en el grupo de recursos de Azure

En el modelo de implementación clásica de Azure, hay una dependencia entre el equilibrador de carga interno de Azure (servicio Azure Load Balancer) y el servicio en la nube. Cada equilibrador de carga interno precisa de un servicio en la nube.

En Azure Resource Manager, cada recurso de Azure debe ubicarse en un grupo de recursos de Azure y esto también es válido para Azure Load Balancer. Sin embargo, no es necesario tener un grupo de recursos de Azure por cada equilibrador de carga de Azure; por ejemplo, un grupo de recursos de Azure puede contener varios equilibradores de carga de Azure. El entorno es más sencillo y flexible. 

### <a name="support-for-sap-multi-sid-scenarios"></a>Soporte técnico para el escenario de varios SID de SAP

En Azure Resource Manager, puede instalar varias instancias de ASCS/SCS de identificador de sistema SAP (SID) en un clúster. Esto es posible gracias a la compatibilidad con las distintas direcciones IP de cada equilibrador de carga interno de Azure.

Si desea usar el modelo de implementación clásica de Azure, siga los procedimientos que se describen en [SAP NetWeaver in Azure: Clustering SAP ASCS/SCS instances by using Windows Server Failover Clustering in Azure with SIOS DataKeeper](https://go.microsoft.com/fwlink/?LinkId=613056) (SAP NetWeaver en Azure: agrupación de clústeres de instancias de ASCS/SCS de SAP mediante Clústeres de conmutación por error de Windows Server en Azure con SIOS DataKeeper).

> [!IMPORTANT]
> Recomendamos encarecidamente usar el modelo de implementación de Azure Resource Manager para las instalaciones de SAP. Ofrece muchas ventajas que no están disponibles en el modelo de implementación clásica. Obtenga más información acerca de los [modelos de implementación de Azure][virtual-machines-azure-resource-manager-architecture-benefits-arm].   
>
>

## <a name="windows-server-failover-clustering"></a><a name="8ecf3ba0-67c0-4495-9c14-feec1a2255b7"></a> Clústeres de conmutación por error de Windows Server
Clústeres de conmutación por error de Windows Server es la base de una instalación de ASCS/SCS de SAP de alta disponibilidad y DBMS en Windows.

Un clúster de conmutación por error es un grupo de 1+n servidores independientes (nodos) que colaboran para aumentar la disponibilidad de aplicaciones y servicios. Si se produce un error de nodo, Clústeres de conmutación por error de Windows Server calcula el número de errores que se pueden producir y mantiene un clúster en buen estado para proporcionar aplicaciones y servicios. Para conseguir clústeres de conmutación por error, puede elegir entre distintos modos de cuórum.

### <a name="quorum-modes"></a><a name="1a3c5408-b168-46d6-99f5-4219ad1b1ff2"></a> Modos de cuórum
Puede elegir entre cuatro modos de cuórum cuando use Clústeres de conmutación por error de Windows Server:

* **Mayoría de nodo**. Cada nodo del clúster puede votar. El clúster funciona solamente con la mayoría de los votos, es decir, con más de la mitad. Se recomienda esta opción para los clústeres con un número impar de nodos. Por ejemplo, 3 nodos de un clúster de 7 nodos pueden presentar un error y el clúster todavía obtendrá la mayoría y se seguirá ejecutando.  
* **Mayoría de disco y nodo**. Cada nodo y disco designado (un testigo de disco) en el almacenamiento del clúster puede votar siempre que esté disponible y comunicándose. El clúster funciona solamente con la mayoría de los votos, es decir, con más de la mitad. Este modo resulta útil en un entorno de clúster con un número par de nodos. Si la mitad de los nodos y el disco están en línea, el clúster permanece en buen estado.
* **Mayoría de recurso compartido de archivos y nodo**. Cada nodo más un recurso compartido de archivos designado (un testigo de recurso de archivos) que el administrador crea puede votar, independientemente de si los nodos y el recurso compartido de archivos están disponibles y comunicándose. El clúster funciona solamente con la mayoría de los votos, es decir, con más de la mitad. Este modo resulta útil en un entorno de clúster con un número par de nodos. Es similar al modo Mayoría de disco y nodo, pero usa un recurso compartido de archivos testigo en lugar de un disco testigo. Este modo es fácil de implementar pero, si el recurso compartido de archivos en sí no es de alta disponibilidad, es posible que se convierta en un único punto de error.
* **Sin mayoría: solo disco**. El clúster tiene cuórum si hay un nodo disponible y comunicándose con un disco específico en el almacenamiento del clúster. Los únicos nodos que pueden unirse al clúster son aquellos que estén comunicándose con ese disco. No se recomienda usar este modo.


## <a name="windows-server-failover-clustering-on-premises"></a><a name="fdfee875-6e66-483a-a343-14bbaee33275"></a> Clústeres de conmutación por error de Windows Server local
En la figura 1 se muestra un clúster de dos nodos. Si se produce un error en la conexión de red entre los nodos pero ambos nodos se mantienen activos y en ejecución, un recurso compartido de archivos o un disco de cuórum determinará el nodo que seguirá proporcionando las aplicaciones y servicios del clúster. El nodo que tiene acceso al recurso compartido de archivos o disco de cuórum es el nodo que garantiza la continuación de los servicios.

Debido a que este ejemplo usa un clúster de dos nodos, usamos el modo de cuórum Mayoría de recurso compartido de archivos y nodo. Otra opción válida es el modo Mayoría de disco y nodo. En un entorno de producción, se recomienda usar un disco de cuórum. Puede usar tecnología de sistema de almacenamiento y red para hacerlo de alta disponibilidad.

![Figura 1: Ejemplo de una configuración de clústeres de conmutación por error de Windows Server para ASCS/SCS de SAP en Azure][sap-ha-guide-figure-1000]

_**Ilustración 1:** Ejemplo de una configuración de clústeres de conmutación por error de Windows Server para ASCS/SCS de SAP en Azure_

### <a name="shared-storage"></a><a name="be21cf3e-fb01-402b-9955-54fbecf66592"></a> Almacenamiento compartido
En la figura 1 también se muestra un clúster de almacenamiento compartido de dos nodos. En un clúster de almacenamiento compartido local, todos los nodos del clúster detectan el almacenamiento compartido. Un mecanismo de bloqueo protege los datos contra daños. Todos los nodos pueden detectar si se produce un error en otro nodo. Si se produce un error en un nodo, el nodo restante asume la propiedad de los recursos de almacenamiento y garantiza la disponibilidad de los servicios.

> [!NOTE]
> No necesita discos compartidos para obtener alta disponibilidad con algunas aplicaciones de DBMS, como SQL Server. SQL Server Always On replica archivos de registro y datos de DBMS desde el disco local de un nodo del clúster hasta el disco local del otro nodo del clúster. En ese caso, la configuración de clúster de Windows no necesita un disco compartido.
>
>

### <a name="networking-and-name-resolution"></a><a name="ff7a9a06-2bc5-4b20-860a-46cdb44669cd"></a> Redes y resolución de nombres
Los equipos cliente se comunican con el clúster a través de una dirección IP virtual y un nombre de host virtual que el servidor DNS proporciona. Los nodos locales y el servidor DNS pueden administrar varias direcciones IP.

En una configuración típica, puede usar dos o más conexiones de red:

* Una conexión dedicada al almacenamiento.
* Una conexión de red interna del clúster para el latido.
* Una red pública que los clientes usan para conectarse al clúster.

## <a name="windows-server-failover-clustering-in-azure"></a><a name="2ddba413-a7f5-4e4e-9a51-87908879c10a"></a> Clústeres de conmutación de Windows Server en Azure
En comparación con las implementaciones de nube privada o sin sistema operativo, Azure Virtual Machines requiere pasos adicionales para configurar Clústeres de conmutación por error de Windows Server. Cuando compila un disco de clúster compartido, deberá establecer varias direcciones IP y nombres de host virtual para la instancia de ASCS/SCS de SAP.

En este artículo se analizan los conceptos clave y los pasos adicionales que se requieren para compilar un clúster de servicios centrales de alta disponibilidad de SAP en Azure. Mostramos cómo configurar la herramienta de terceros SIOS DataKeeper y configurar el equilibrador de carga interno de Azure. Puede usar estas herramientas para crear un clúster de conmutación por error de Windows con un testigo de recurso compartido de archivos en Azure.

![Ilustración 2: Configuración de clústeres de conmutación por error de Windows Server en Azure sin un disco compartido][sap-ha-guide-figure-1001]

_**Ilustración 2:** Configuración de clústeres de conmutación por error de Windows Server en Azure sin un disco compartido_

### <a name="shared-disk-in-azure-with-sios-datakeeper"></a><a name="1a464091-922b-48d7-9d08-7cecf757f341"></a> Disco compartido en Azure con SIOS DataKeeper
Se necesita almacenamiento compartido de clúster para una instancia de ASCS/SCS de SAP de alta disponibilidad. A partir de septiembre de 2016, Azure no ofrece almacenamiento compartido que puede usar para crear un clúster de almacenamiento compartido. Puede usar el software de terceros SIOS DataKeeper Cluster Edition para crear un almacenamiento reflejado que simula el almacenamiento compartido de clúster. La solución SIOS proporciona replicación sincrónica de datos en tiempo real. Este es el procedimiento para crear un recurso de disco compartido para un clúster:

1. Asocie un disco adicional a cada una de las máquinas virtuales de una configuración de clúster de Windows.
2. Ejecute SIOS DataKeeper Cluster Edition en ambos nodos de la máquina virtual.
3. Configure SIOS DataKeeper Cluster Edition de forma que refleje el contenido del volumen del disco adicional asociado desde la máquina virtual de origen al volumen del disco adicional asociado de la máquina virtual de destino. SIOS DataKeeper abstrae los volúmenes locales de origen y de destino, y los presenta a Clústeres de conmutación por error de Windows Server como un disco compartido.

Obtenga más información sobre [SIOS DataKeeper](https://us.sios.com/products/datakeeper-cluster/).

![Ilustración 3: Configuración de clústeres de conmutación por error de Windows Server en Azure con SIOS DataKeeper][sap-ha-guide-figure-1002]

_**Ilustración 3:** Configuración de clústeres de conmutación por error de Windows Server en Azure con SIOS DataKeeper_

> [!NOTE]
> No necesita discos compartidos para obtener alta disponibilidad con algunos productos de DBMS, como SQL Server. SQL Server Always On replica archivos de registro y datos de DBMS desde el disco local de un nodo del clúster hasta el disco local del otro nodo del clúster. En este caso, la configuración de clúster de Windows no necesita un disco compartido.
>
>

### <a name="name-resolution-in-azure"></a><a name="44641e18-a94e-431f-95ff-303ab65e0bcb"></a> Resolución de nombres en Azure
La plataforma en la nube de Azure no ofrece la opción de configurar direcciones IP virtuales, como direcciones IP flotantes. Necesita una solución alternativa para configurar una dirección IP virtual que se comunique con el recurso de clúster en la nube.
Azure tiene un equilibrador de carga interno en el servicio Azure Load Balancer. Con el equilibrador de carga interno, los clientes se comunican con el clúster a través de la dirección IP virtual del clúster.
Necesita implementar el equilibrador de carga interno en el grupo de recursos que contiene los nodos del clúster. Luego, configure todas las reglas de enrutamiento de puerto necesarias con los puertos de sondeo del equilibrador de carga interno.
Los clientes se pueden conectar por medio del nombre de host virtual. El servidor DNS resuelve la dirección IP del clúster y el equilibrador de carga interno controla el enrutamiento de puerto al nodo activo del clúster.

## <a name="sap-netweaver-high-availability-in-azure-infrastructure-as-a-service-iaas"></a><a name="2e3fec50-241e-441b-8708-0b1864f66dfa"></a> Alta disponibilidad de SAP NetWeaver en infraestructura como servicio (IaaS) de Azure
Para obtener alta disponibilidad en las aplicaciones de SAP, como para los componentes de software de SAP, necesita proteger los componentes siguientes:

* Instancia de servidores de aplicaciones de SAP
* instancia de SAP ASCS/SCS,
* servidor DBMS

Para obtener más información sobre cómo proteger los componentes SAP en escenarios de alta disponibilidad, consulte [Planeamiento e implementación de Azure Virtual Machines para SAP NetWeaver][planning-guide-11].

### <a name="high-availability-sap-application-server"></a><a name="93faa747-907e-440a-b00a-1ae0a89b1c0e"></a> Alta disponibilidad en los servidores de aplicaciones de SAP
Por lo general, no se necesita una solución de alta disponibilidad específica para las instancias de diálogo y servidores de aplicaciones de SAP. Obtiene alta disponibilidad mediante redundancia y configurará varias instancias de diálogo en distintas instancias de Azure Virtual Machines. Debe tener, como mínimo, 2 instancias de aplicaciones de SAP instaladas en 2 instancias de Azure Virtual Machines.

![Ilustración 4: Alta disponibilidad en los servidores de aplicaciones de SAP][sap-ha-guide-figure-2000]

_**Ilustración 4:** Alta disponibilidad en los servidores de aplicaciones de SAP_

Debe colocar todas las máquinas virtuales que hospedan instancias de servidores de aplicaciones de SAP en el mismo conjunto de disponibilidad de Azure. Un conjunto de disponibilidad de Azure garantiza lo siguiente:

* Todas las máquinas virtuales forman parte del mismo dominio de actualización. Por ejemplo, un dominio de actualización garantiza que las máquinas virtuales no se actualicen al mismo tiempo durante un tiempo de inactividad de mantenimiento planeado.
* Todas las máquinas virtuales forman parte del mismo dominio de error. Por ejemplo, un dominio de error garantiza que las máquinas virtuales estén implementadas de tal manera que ningún único punto de error afecte la disponibilidad de todas las máquinas virtuales.

Obtenga más información sobre cómo [administrar la disponibilidad de máquinas virtuales][../manage-availability.md].

Solo disco no administrado: Como la cuenta de almacenamiento de Azure es un punto único de error potencial, es importante tener, como mínimo, 2 cuentas de almacenamiento de Azure, en los cuales hay distribuidas, al menos, 2 máquinas virtuales. En una configuración ideal, los discos de cada máquina virtual que ejecuta una instancia de diálogo de SAP estarían implementados en una cuenta de almacenamiento distinta.

### <a name="high-availability-sap-ascsscs-instance"></a><a name="f559c285-ee68-4eec-add1-f60fe7b978db"></a> Instancia de ASCS/SCS de SAP de alta disponibilidad
La figura 5 es un ejemplo de una instancia de ASCS/SCS de SAP de alta disponibilidad.

![Ilustración 5: Instancia de ASCS/SCS de SAP de alta disponibilidad][sap-ha-guide-figure-2001]

_**Ilustración 5:** Instancia de ASCS/SCS de SAP de alta disponibilidad_

#### <a name="sap-ascsscs-instance-high-availability-with-windows-server-failover-clustering-in-azure"></a><a name="b5b1fd0b-1db4-4d49-9162-de07a0132a51"></a> Alta disponibilidad de instancia de ASCS/SCS de SAP con Clústeres de conmutación por error de Windows Server en Azure
En comparación con las implementaciones de nube privada o sin sistema operativo, Azure Virtual Machines requiere pasos adicionales para configurar Clústeres de conmutación por error de Windows Server. A fin de compilar un clúster de conmutación por error de Windows, necesita un disco de clúster compartido, varias direcciones IP, varios nombres de host virtual y un equilibrador de carga interno de Azure para la agrupación en clústeres de una instancia de ASCS/SCS de SAP. Analizaremos esto con más detalle más adelante en el artículo.

![Figura 6: Clústeres de conmutación por error de Windows Server para una configuración de ASCS/SCS de SAP en Azure con SIOS DataKeeper][sap-ha-guide-figure-1002]

_**Figura 6:** Clústeres de conmutación por error de Windows Server para una configuración de ASCS/SCS de SAP en Azure con SIOS DataKeeper_

### <a name="high-availability-dbms-instance"></a><a name="ddd878a0-9c2f-4b8e-8968-26ce60be1027"></a>Alta disponibilidad para la instancia de DBMS
DBMS también es un único punto de contacto de un sistema SAP. Debe protegerlo con una solución de alta disponibilidad. La figura 7 muestra una solución de alta disponibilidad de SQL Server AlwaysOn en Azure mediante el uso de Clústeres de conmutación por error de Windows Server y el equilibrador de carga interno de Azure. SQL Server AlwaysOn replica los archivos de datos y de registro de DBMS con su propia replicación DBMS. En este caso, no se necesitan discos compartidos de clúster, lo que simplifica toda la configuración.

![Figura 7: Ejemplo de DBMS con SAP de alta disponibilidad con SQL Server AlwaysOn][sap-ha-guide-figure-2003]

_**Figura 7:** Ejemplo de DBMS con SAP de alta disponibilidad con SQL Server AlwaysOn_

Para más información sobre cómo agrupar el clústeres SQL Server en Azure con el modelo de implementación de Azure Resource Manager, consulte estos artículos:

* [Configuración manual de grupos de disponibilidad AlwaysOn en Azure Virtual Machines con Resource Manager][virtual-machines-windows-portal-sql-alwayson-availability-groups-manual]
* [Configuración de un equilibrador de carga interno para un grupo de disponibilidad AlwaysOn en Azure][virtual-machines-windows-portal-sql-alwayson-int-listener]

## <a name="end-to-end-high-availability-deployment-scenarios"></a><a name="045252ed-0277-4fc8-8f46-c5a29694a816"></a> Escenarios de implementación completa de alta disponibilidad

### <a name="deployment-scenario-using-architectural-template-1"></a>Escenario de implementación con la plantilla 1 de arquitectura

En la figura 8 se muestra un ejemplo de una arquitectura de SAP NetWeaver de alta disponibilidad en Azure para **UN** sistema SAP. Este escenario se configura como se indica a continuación:

- Se usa un clúster dedicado para la instancia de ASCS/SCS de SAP.
- Se usa un clúster dedicado para la instancia de DBMS.
- Se implementan instancias de servidores de aplicaciones SAP en máquinas virtuales dedicadas propias.

![Figura 8: Plantilla 1 de arquitectura de alta disponibilidad de SAP con un clúster dedicado para ASCS/SCS y DBMS][sap-ha-guide-figure-2004]

_**Figura 8:** Plantilla 1 de arquitectura de alta disponibilidad de SAP con un clúster dedicado para ASCS/SCS y DBMS_

### <a name="deployment-scenario-using-architectural-template-2"></a>Escenario de implementación con la plantilla 2 de arquitectura

En la figura 9 se muestra un ejemplo de una arquitectura de SAP NetWeaver de alta disponibilidad en Azure para **UN** sistema SAP. Este escenario se configura como se indica a continuación:

- Se usa un clúster dedicado **TANTO** para la instancia de ASCS/SCS de SAP como para la de DBMS.
- Se implementan instancias de servidores de aplicaciones SAP en máquinas virtuales dedicadas propias.

![Figura 9: Plantilla 2 de arquitectura de alta disponibilidad de SAP, con un clúster dedicado para ASCS/SCS y otro para la instancia de DBMS][sap-ha-guide-figure-2005]

_**Figura 9:** Plantilla 2 de arquitectura de alta disponibilidad de SAP, con un clúster dedicado para ASCS/SCS y otro para la instancia de DBMS_

### <a name="deployment-scenario-using-architectural-template-3"></a>Escenario de implementación con la plantilla 3 de arquitectura

En la figura 10 se muestra un ejemplo de una arquitectura de SAP NetWeaver de alta disponibilidad en Azure para **DOS** sistemas SAP, con &lt;SID1&gt; y &lt;SID2&gt;. Este escenario se configura como se indica a continuación:

- Un clúster dedicado se usa **TANTO** para la instancia SID1 de ASCS/SCS *como* para la instancia SID2 de ASCS/SCS de SAP (un clúster).
- Se usa un clúster dedicado para SID 1 de DBMS y otro para SID2 de DBMS (dos clústeres).
- Las instancias de servidores de aplicaciones SAP del SID1 del sistema SAP tienen su propia máquina virtual dedicada.
- Las instancias de servidores de aplicaciones SAP del SID2 del sistema SAP tienen su propia máquina virtual dedicada.

![Figura 10: Plantilla 3 de arquitectura de alta disponibilidad de SAP, con un clúster dedicado para las diferentes instancias de ASCS/SCS][sap-ha-guide-figure-6003]

_**Figura 10:** Plantilla 3 de arquitectura de alta disponibilidad de SAP, con un clúster dedicado para las diferentes instancias de ASCS/SCS_

## <a name="prepare-the-infrastructure"></a><a name="78092dbe-165b-454c-92f5-4972bdbef9bf"></a> Preparación de la infraestructura

### <a name="prepare-the-infrastructure-for-architectural-template-1"></a>Preparación de la infraestructura de la plantilla 1 de arquitectura
Las plantillas de Azure Resource Manager para SAP ayudan a simplificar la implementación de los recursos necesarios.

Las plantillas de 3 niveles de Azure Resource Manager también admiten escenarios de alta disponibilidad, como la plantilla 1 de arquitectura, que tiene 2 clústeres. Cada clúster es un único punto de error de SAP para ASCS/SCS de SAP y DBMS.

Aquí puede obtener las plantillas de Azure Resource Manager para el escenario de ejemplo que se describe en este artículo:

* [Imagen de Azure Marketplace](https://github.com/Azure/azure-quickstart-templates/)  
* [Imagen de Azure Marketplace con Managed Disks](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md)  
* [Imagen personalizada](https://github.com/Azure/azure-quickstart-templates/)
* [Imagen personalizada con Managed Disks](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-md)

Para preparar la plantilla 1 de arquitectura, siga estos pasos:

- En Azure Portal, en la hoja **Parámetros** del cuadro **SYSTEMAVAILABILITY**, seleccione **Alta disponibilidad**.

  ![Figura 11: Configuración de los parámetros de Azure Resource Manager para alta disponibilidad de SAP][sap-ha-guide-figure-3000]

_**Figura 11:** Configuración de los parámetros de Azure Resource Manager para alta disponibilidad de SAP_


  Las plantillas crean:

  * **Máquinas virtuales**:
    * Máquinas virtuales de Servidor de aplicaciones de SAP: <*SIDSistemaSAP*>-di-<*Número*>
    * Máquinas virtuales de clúster de ASCS/SCS: <*SIDSistemaSAP*>-ascs-<*Número*>
    * Un clúster de DBMS: <*SIDSistemaSAP*>-db-<*Número*>

  * **Tarjetas de red para todas las máquinas virtuales con direcciones IP asociadas**:
    * <*SIDSistemaSAP*&gt;-nic-di-&lt;*Número*>
    * <*SIDSistemaSAP*&gt;-nic-ascs-&lt;*Número*>
    * <*SIDSistemaSAP*&gt;-nic-db-&lt;*Número*>

  * **Cuentas de Azure Storage (solo discos no administrados)**

  * **Grupos de disponibilidad** para:
    * Máquinas virtuales de Servidor de aplicaciones de SAP: <*SIDSistemaSAP*>-avset-di
    * Máquinas virtuales de clúster de ASCS/SCS de SAP: <*SIDSistemaSAP*>-avset-ascs
    * Máquinas virtuales de clúster de DBMS: <*SIDSistemaSAP*>-avset-db

  * **Equilibrador de carga interno de Azure**:
    * Con todos los puertos para la dirección IP y la instancia de ASCS/SCS <*SIDSistemaSAP*>-lb-ascs
    * Con todos los puertos para la dirección IP y DBMS de SQL Server <*SIDSistemaSAP*>-lb-db

  * **Grupo de seguridad de red**: &lt;*SIDSistemaSAP*&gt;-nsg-ascs-0  
    * Con un puerto de Protocolo de escritorio remoto (RDP) externo abierto para la máquina virtual <*SIDSistemaSAP*>-ascs-0

> [!NOTE]
> De manera predeterminada, todas las direcciones IP de las tarjetas de red y los equilibradores de carga internos de Azure son **dinámicos**. Cámbielas a direcciones IP **estáticas**. Esto se describe más adelante en el artículo.
>
>

### <a name="deploy-virtual-machines-with-corporate-network-connectivity-cross-premises-to-use-in-production"></a><a name="c87a8d3f-b1dc-4d2f-b23c-da4b72977489"></a> Implementación de máquinas virtuales con conectividad de red corporativa (entre locales) para uso en producción
Para los sistemas de producción de SAP, implemente máquinas virtuales de Azure con conectividad de red corporativa mediante el uso de VPN de sitio a sitio de Azure o Azure ExpressRoute.

> [!NOTE]
> Puede usar la instancia de Azure Virtual Network. La red virtual y la subred ya se crearon y están preparadas.
>
>

1. En Azure Portal, en la hoja **Parámetros** del cuadro **NEWOREXISTINGSUBNET**, seleccione **Existente**.
2. En el cuadro **SUBNETID**, agregue la cadena completa de SubnetID de la red de Azure, donde planea implementar las máquinas virtuales de Azure.
3. Ejecute este comando de PowerShell para obtener una lista de todas las subredes de red de Azure:

   ```powershell
   (Get-AzVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets
   ```

   El campo **ID** muestra el valor de **SUBNETID**.
4. Ejecute este comando de PowerShell para obtener una lista de todos los valores de **SUBNETID**:

   ```powershell
   (Get-AzVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets.Id
   ```

   El aspecto del identificador **SUBNETID** es similar al siguiente:

   ```
   /subscriptions/<SubscriptionId>/resourceGroups/<VPNName>/providers/Microsoft.Network/virtualNetworks/azureVnet/subnets/<SubnetName>
   ```

### <a name="deploy-cloud-only-sap-instances-for-test-and-demo"></a><a name="7fe9af0e-3cce-495b-a5ec-dcb4d8e0a310"></a> Implementación solo en la nube de instancias de SAP para prueba o demostración
Puede implementar el sistema de SAP de alta disponibilidad en un modelo de implementación solo en la nube. Este tipo de implementación se usa principalmente para demostrar o probar casos de uso. No es idóneo para los casos de uso de producción.

- En Azure Portal, en la hoja **Parámetros** del cuadro **NEWOREXISTINGSUBNET**, seleccione **Nuevo**. Deje vacío el campo **SUBNETID**.

  La plantilla de Azure Resource Manager para SAP crea automáticamente la subred y red virtual de Azure.

> [!NOTE]
> También debe implementar, como mínimo, una máquina virtual dedicada para Active Directory y DNS en la misma instancia de Azure Virtual Network. La plantilla no crea estas máquinas virtuales.
>
>


### <a name="prepare-the-infrastructure-for-architectural-template-2"></a>Preparación de la infraestructura de la plantilla 2 de arquitectura

Puede usar estas plantillas de Azure Resource Manager para simplificar la implementación de los recursos de la infraestructura necesarios para la plantilla 2 de arquitectura de SAP.

Aquí puede obtener las plantillas de Azure Resource Manager para este escenario de implementación:

* [Imagen de Azure Marketplace](https://github.com/Azure/azure-quickstart-templates/)  
* [Imagen de Azure Marketplace con Managed Disks](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-converged-md)  
* [Imagen personalizada](https://github.com/Azure/azure-quickstart-templates/)
* [Imagen personalizada con Managed Disks](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-converged-md)


### <a name="prepare-the-infrastructure-for-architectural-template-3"></a>Preparación de la infraestructura de la plantilla 3 de arquitectura

Puede preparar la infraestructura y configurar SAP para **varios SID**. Por ejemplo, puede agregar una instancia de ASCS/SCS de SAP adicional en una configuración de clúster *existente*. Para obtener más información, consulte [Configuración de una instancia adicional de ASCS/SCS de SAP en una configuración de clúster existente para crear una configuración de varios SID de SAP: Azure Resource Manager][sap-ha-multi-sid-guide].

Si desea crear un nuevo clúster de varios SID, puede usar las [plantillas de inicio rápido en GitHub](https://github.com/Azure/azure-quickstart-templates) para varios SID.
Para crear un nuevo clúster de varios SID, debe implementar las tres plantillas siguientes:

* [Plantilla de ASCS/SCS](#ASCS-SCS-template)
* [Plantilla de base de datos](#database-template)
* [Plantilla de servidores de aplicaciones](#application-servers-template)

Las siguientes secciones contienen más detalles sobre las plantillas y los parámetros que debe proporcionar en las plantillas.

#### <a name="ascsscs-template"></a><a name="ASCS-SCS-template"></a>Plantilla de ASCS/SCS

La plantilla de ASCS/SCS permite implementar dos máquinas virtuales que se pueden usar para crear un clúster de conmutación por error de Windows Server que hospeda varias instancias de ASCS/SCS.

Para configurar la plantilla de varios SID de ASCS/SCS, en la [plantilla de varios SID de ASCS/SCS][sap-templates-3-tier-multisid-xscs-marketplace-image] o en la [plantilla de varios SID de ASCS/SCS con Managed Disks][sap-templates-3-tier-multisid-xscs-marketplace-image-md], especifique los valores para los parámetros siguientes:

  - **Prefijo de recurso**.  Establezca el prefijo de recurso, que se utiliza para prefijar todos los recursos que se hayan creado durante la implementación. Dado que los recursos no pertenecen a un único sistema SAP, el prefijo del recurso no es el SID de un sistema SAP.  El prefijo debe tener entre **tres y seis caracteres**.
  - **Tipo de pila**. Seleccione el tipo de la pila del sistema SAP. Dependiendo del tipo de la pila, Azure Load Balancer tendrá una (solo ABAP o JAVA) o dos (ABAP + JAVA) direcciones IP privadas por cada sistema SAP.
  -  **Tipo de sistema operativo**. Seleccione el sistema operativo de las máquinas virtuales.
  -  **Recuento de sistema SAP**. Seleccione número de sistemas SAP que desea instalar en este clúster.
  -  **Disponibilidad del sistema**. Seleccione **Alta disponibilidad**.
  -  **Nombre de usuario y contraseña del administrador**. Cree un nuevo usuario que pueda usarse para iniciar sesión en la máquina.
  -  **Subred nueva o existente**. Establezca si es necesario crear una red virtual y subred nuevas o si se debe usar una subred existente. Si ya tiene una red virtual conectada a la red local, seleccione la **existente**.
  -  **Identificador de subred**. Si quiere implementar la máquina virtual en una red virtual existente en la que tiene una subred definida a la que se debe asignar la máquina virtual, asigne un nombre al identificador de esa subred específica. El identificador suele tener este formato: /subscriptions/<*id. de suscripción*>/resourceGroups/<*nombre del grupo de recursos*>/providers/Microsoft.Network/virtualNetworks/<*nombre de la red virtual*>/subnets/<*nombre la de subred*>

La plantilla implementa una instancia de Azure Load Balancer que admite varios sistemas SAP.

- Las instancias de ASCS se configuran según el número de instancia 00, 10, 20...
- Las instancias de SCS se configuran según el número de instancia 01, 11, 21...
- Las instancias de ASCS Enqueue Replication Server (ERS) (solo Linux) se configuran según el número de instancia 02, 12, 22, etc.
- Las instancias de SCS ERS (solo Linux) se configuran según el número de instancia 03, 13, 23...

El equilibrador de carga contiene 1 (2 para Linux) VIP, 1 VIP para ASCS/SCS y 1 VIP para ERS (solo Linux).

En la lista siguiente se incluyen todas las reglas de equilibrio de carga (donde x es el número del sistema SAP, por ejemplo, 1, 2, 3, etc.):
- Puertos específicos de Windows para cada sistema SAP: 445, 5985
- Puertos ASCS (número de instancia x0): 32x0, 36x0, 39x0, 81x0, 5x013, 5x014, 5x016
- Puertos SCS (número de instancia x1): 32x1, 33x1, 39x1, 81x1, 5x113, 5x114, 5x116
- Puertos ASCS ERS en Linux (número de instancia x2): 33x2, 5x213, 5x214, 5x216
- Puertos SCS ERS en Linux (número de instancia x3): 33x3, 5x313, 5x314, 5x316

El equilibrador de carga se configurará para usar los siguientes puertos de sondeo (donde x es el número del sistema SAP, por ejemplo, 1, 2, 3,...):
- Puerto de sondeo del equilibrador de carga interno de ASCS/SCS: 620x0
- Puerto de sondeo del equilibrador de carga interno de ERS (solo para Linux): 621x2

#### <a name="database-template"></a><a name="database-template"></a> Plantilla de base de datos

La plantilla de la base de datos implementa uno o dos máquinas virtuales que puede usar para instalar el sistema de administración de bases de datos relacionales (RDBMS) de un sistema SAP. Por ejemplo, si ha implementado una plantilla de ASCS/SCS para 5 sistemas SAP, debe implementar esta plantilla cinco veces.

Para configurar la plantilla de varios SID de base de datos, en la [plantilla de varios SID de base de datos][sap-templates-3-tier-multisid-db-marketplace-image] o en la [plantilla de varios SID de base de datos con Managed Disks][sap-templates-3-tier-multisid-db-marketplace-image-md], especifique los valores para los parámetros siguientes:

- **Identificador de sistema SAP**. Escriba el identificador del sistema SAP que se va a instalar. El identificador se utilizará como prefijo para los recursos que se implementen.
- **Tipo de sistema operativo**. Seleccione el sistema operativo de las máquinas virtuales.
- **Dbtype**. Seleccione el tipo de base de datos que desea instalar en el clúster. Seleccione **SQL** si desea instalar Microsoft SQL Server. Seleccione **HANA** si tiene previsto instalar SAP HANA en las máquinas virtuales. Asegúrese de seleccionar el tipo de sistema operativo correcto: **Windows** para SQL y una distribución de Linux para HANA. La instancia de Azure Load Balancer que está conectada a las máquinas virtuales se configurará para admitir el tipo de base de datos seleccionado:
  * **SQL**. El equilibrador de carga equilibrará la carga del puerto 1433. Asegúrese de que utiliza este puerto para la configuración de SQL Server Always On.
  * **HANA**. El equilibrador de carga equilibrará la carga de los puertos 35015 y 35017. Asegúrese de instalar SAP HANA con el número de instancia **50**.
  El equilibrador de carga usará el puerto de sondeo 62550.
- **Tamaño del sistema SAP**. La cantidad de sistemas SAP que proporcionará el sistema nuevo. Si no está seguro de cuántos SAPS necesitará el sistema, consulte con el integrador de sistemas o el asociado tecnológico de SAP.
- **Disponibilidad del sistema**. Seleccione **Alta disponibilidad**.
- **Nombre de usuario y contraseña del administrador**. Cree un nuevo usuario que pueda usarse para iniciar sesión en la máquina.
- **Identificador de subred**. Escriba el identificador de la subred que usó durante la implementación de la plantilla de ASCS/SCS o el identificador de la subred que se creó como parte de la implementación de esta plantilla.

#### <a name="application-servers-template"></a><a name="application-servers-template"></a>Plantilla de servidores de aplicaciones

La plantilla de servidores de aplicaciones permite implementar dos o más máquinas virtuales que se pueden usar como instancias de servidores de aplicaciones SAP en un sistema SAP. Por ejemplo, si ha implementado una plantilla de ASCS/SCS para 5 sistemas SAP, debe implementar esta plantilla cinco veces.

Para configurar la plantilla de varios SID de servidores de aplicaciones, en la [plantilla de varios SID de servidores de aplicaciones][sap-templates-3-tier-multisid-apps-marketplace-image] o en la [plantilla de varios SID de servidores de aplicaciones con Managed Disks][sap-templates-3-tier-multisid-apps-marketplace-image-md], especifique los valores para los parámetros siguientes:

  -  **Identificador de sistema SAP**. Escriba el identificador del sistema SAP que se va a instalar. El identificador se utilizará como prefijo para los recursos que se implementen.
  -  **Tipo de sistema operativo**. Seleccione el sistema operativo de las máquinas virtuales.
  -  **Tamaño del sistema SAP**. La cantidad de SAPS que proporcionará el sistema nuevo. Si no está seguro de cuántos SAPS necesitará el sistema, consulte con el integrador de sistemas o el asociado tecnológico de SAP.
  -  **Disponibilidad del sistema**. Seleccione **Alta disponibilidad**.
  -  **Nombre de usuario y contraseña del administrador**. Cree un nuevo usuario que pueda usarse para iniciar sesión en la máquina.
  -  **Identificador de subred**. Escriba el identificador de la subred que usó durante la implementación de la plantilla de ASCS/SCS o el identificador de la subred que se creó como parte de la implementación de esta plantilla.


### <a name="azure-virtual-network"></a><a name="47d5300a-a830-41d4-83dd-1a0d1ffdbe6a"></a> Azure Virtual Network
En nuestro ejemplo, el espacio de direcciones de la máquina virtual de Azure es 10.0.0.0/16. Hay una subred llamada **Subnet**, con un intervalo de direcciones de 10.0.0.0/24. Todas las máquinas virtuales y los equilibradores de carga internos están implementados en esta red virtual.

> [!IMPORTANT]
> No haga ningún cambio en la configuración de red dentro del sistema operativo invitado. Esto incluye direcciones IP, servidores DNS y subred. Ajuste toda la configuración de red de Azure. El servicio Protocolo de configuración dinámica de host (DHCP) propaga la configuración.
>
>

### <a name="dns-ip-addresses"></a><a name="b22d7b3b-4343-40ff-a319-097e13f62f9e"></a> Direcciones IP de DNS

Para configurar las direcciones IP de DNS que se requieren, realice los pasos siguientes.

1. En Azure Portal, en la hoja **Servidores DNS**, asegúrese de que la opción **Servidores DNS** de la red virtual está establecida en **DNS personalizado**.
2. Seleccione la configuración según el tipo de red que tiene. Para obtener más información, consulte los siguientes recursos:
   * agregue las direcciones IP de los servidores DNS locales.  
   Puede extender los servidores DNS locales a las máquinas virtuales que se ejecutan en Azure. En ese escenario, puede agregar las direcciones IP de las máquinas virtuales de Azure en las que ejecuta el servidor DNS.
   * Para implementaciones de máquina virtual aisladas en Azure: implemente una máquina virtual adicional en la misma instancia de Virtual Network que actúa como servidor DNS. Agregue las direcciones IP de las máquinas virtuales de Azure que configuró para ejecutar el servicio DNS.

   ![Figura 12: Configuración de servidores DNS para Azure Virtual Network][sap-ha-guide-figure-3001]

   _**Figura 12:** Configuración de servidores DNS para Azure Virtual Network_

   > [!NOTE]
   > Si cambia las direcciones IP de los servidores DNS, debe reiniciar las máquinas virtuales de Azure para aplicar el cambio y propagar los nuevos servidores DNS.
   >
   >

En este ejemplo, el servicio DNS está instalado y configurado en estas máquinas virtuales de Windows:

| Rol de máquina virtual | Nombre de host de máquina virtual | Nombre de tarjeta de red | Dirección IP estática |
| --- | --- | --- | --- |
| Primer servidor DNS |domcontr-0 |pr1-nic-domcontr-0 |10.0.0.10 |
| Segundo servidor DNS |domcontr-1 |pr1-nic-domcontr-1 |10.0.0.11 |

### <a name="host-names-and-static-ip-addresses-for-the-sap-ascsscs-clustered-instance-and-dbms-clustered-instance"></a><a name="9fbd43c0-5850-4965-9726-2a921d85d73f"></a> Nombres de host y direcciones IP estáticas para la instancia en clúster de ASCS/SCS de SAP y la instancia de clúster de DBMS

Para la implementación local, necesita estas direcciones IP y nombres de host reservados:

| Rol de nombre de host virtual | Nombre de host virtual | Dirección IP estática virtual |
| --- | --- | --- |
| Primer nombre de host virtual de clúster de ASCS/SCS de SAP (para la administración del clúster) |pr1-ascs-vir |10.0.0.42 |
| Nombre de host virtual de la instancia de ASCS/SCS de SAP |pr1-ascs-sap |10.0.0.43 |
| Segundo nombre de host virtual de clúster de DBMS de SAP (administración del clúster) |pr1-dbms-vir |10.0.0.32 |

Cuando crea el clúster, crea los nombres de host virtual **pr1-ascs-vir** y **pr1-dbms-vir**, además de las direcciones IP asociadas que administrar el clúster mismo. Para más información sobre cómo hacerlo, consulte [Recopilación de nodos del clúster en la configuración de clúster][sap-ha-guide-8.12.1].

Puede crear manualmente los otros 2 nombres de host virtual, **pr1-ascs-sap** y **pr1-dbms-sap**, y las direcciones IP asociadas en el servidor DNS. La instancia de ASCS/SCS de SAP en clúster y la instancia de DBMS en clúster usan estos recursos. Para más información sobre cómo hacerlo, vea [Creación de un nombre de host virtual para la instancia de ASCS/SCS de SAP en clúster][sap-ha-guide-9.1.1].

### <a name="set-static-ip-addresses-for-the-sap-virtual-machines"></a><a name="84c019fe-8c58-4dac-9e54-173efd4b2c30"></a> Configuración de direcciones IP estáticas para las máquinas virtuales SAP
Después de implementar las máquinas virtuales que usará en el clúster, deberá establecer direcciones IP estáticas para todas las máquinas virtuales. Debe hacerlo en la configuración de Azure Virtual Network y no en el sistema operativo invitado.

1. En Azure Portal, seleccione **Grupo de recursos** > **Tarjeta de red** > **Configuración** > **Dirección IP**.
2. En la hoja **Direcciones IP**, en **Asignación**, seleccione **Estática**. En el cuadro **Dirección IP**, escriba la dirección IP que desea usar.

   > [!NOTE]
   > Si cambia la dirección IP de la tarjeta de red, deberá reiniciar las máquinas virtuales de Azure para aplicar el cambio.  
   >
   >

   ![Figura 13: Establecimiento de direcciones IP estáticas para la tarjeta de red de cada máquina virtual][sap-ha-guide-figure-3002]

   _**Figura 13:** Establecimiento de direcciones IP estáticas para la tarjeta de red de cada máquina virtual_

   Repita este paso para todas las interfaces de red, es decir, para todas las máquinas virtuales, incluidas las que desea usar para Active Directory/servicio DNS.

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

### <a name="set-a-static-ip-address-for-the-azure-internal-load-balancer"></a><a name="7a8f3e9b-0624-4051-9e41-b73fff816a9e"></a> Configuración de una dirección IP estática para el equilibrador de carga interno de Azure

La plantilla de Azure Resource Manager para SAP crea un equilibrador de carga interno de Azure que se usa para el clúster de instancia de ASCS/SCS de SAP y el clúster de DBMS.

> [!IMPORTANT]
> La dirección IP del nombre de host virtual de la instancia de ASCS/SCS de SAP es la misma dirección IP del equilibrador de carga interno de ASCS/SCS de SAP, **pr1-lb-ascs**.
> La dirección IP del nombre virtual de la instancia de DBMS es la misma dirección IP del equilibrador de carga interno de DBMS, **pr1-lb-dbms**.
>
>

Para configurar una dirección IP estática para el equilibrador de carga interno de Azure, siga estos pasos:

1. En la implementación inicial se establece la dirección IP del equilibrador de carga interno como **Dinámica**. En la hoja **Direcciones IP** de Azure Portal, en **Asignación**, seleccione **Estática**.
2. Establezca la dirección IP del equilibrador de carga interno **pr1-lb-ascs** en la dirección IP del nombre de host virtual de la instancia de ASCS/SCS de SAP.
3. Establezca la dirección IP del equilibrador de carga interno **pr1-lb-dbms** en la dirección IP del nombre de host virtual de la instancia de DBMS.

   ![Figura 14: Establecimiento de direcciones IP estáticas para el equilibrador de carga interno de la instancia de ASCS/SCS de SAP][sap-ha-guide-figure-3003]

   _**Figura 14:** Establecimiento de direcciones IP estáticas para el equilibrador de carga interno de la instancia de ASCS/SCS de SAP_

En este ejemplo, aparecen 2 equilibradores de carga internos de Azure que tienen estas direcciones IP estáticas:

| Rol del equilibrador de carga interno de Azure | Nombre del equilibrador de carga interno de Azure | Dirección IP estática |
| --- | --- | --- |
| Equilibrador de carga interno de instancia de ASCS/SCS de SAP |pr1-lb-ascs |10.0.0.43 |
| Equilibrador de carga interno de DBMS de SAP |pr1-lb-dbms |10.0.0.33 |


### <a name="default-ascsscs-load-balancing-rules-for-the-azure-internal-load-balancer"></a><a name="f19bd997-154d-4583-a46e-7f5a69d0153c"></a> Reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure

La plantilla de Azure Resource Manager para SAP crea los puertos que necesita:
* Una instancia de ASCS ABAP con el número de instancia predeterminado **00**
* Una instancia de SCS de Java con el número de instancia predeterminado **01**

Cuando instala la instancia de ASCS/SCS de SAP, debe usar el número de instancia predeterminado **00** para la instancia de ASCS ABAP y el número de instancia predeterminado **01** para la instancia de SCS de Java.

Después, cree estos puntos de conexión de equilibrio de carga interno necesarios para los puertos de SAP NetWeaver.

Para crear estos puntos de conexión de equilibrio de carga interno necesarios, créelos para los puertos de ASCS ABAP de SAP NetWeaver:

| Nombre de regla de equilibrio de carga/servicio | Números de puerto predeterminados | Puertos concretos para (instancia de ASCS con el número de instancia 00) (ERS con 10) |
| --- | --- | --- |
| Enqueue Server/ *lbrule3200* |32<*NúmeroInstancia*> |3200 |
| ABAP Message Server/ *lbrule3600* |36<*NúmeroInstancia*> |3600 |
| Internal ABAP Message/ *lbrule3900* |39<*NúmeroInstancia*> |3900 |
| Message Server HTTP/ *Lbrule8100* |81<*NúmeroInstancia*> |8100 |
| SAP Start Service ASCS HTTP/ *Lbrule50013* |5<*NúmeroInstancia*>13 |50013 |
| SAP Start Service ASCS HTTPS/ *Lbrule50014* |5<*NúmeroInstancia*>14 |50014 |
| Enqueue Replication/ *Lbrule50016* |5<*NúmeroInstancia*>16 |50016 |
| SAP Start Service ERS HTTP/ *Lbrule51013* |5<*NúmeroInstancia*>13 |51013 |
| SAP Start Service ERS HTTP/ *Lbrule51014* |5<*NúmeroInstancia*>14 |51014 |
| Win RM/ *Lbrule5985* | |5985 |
| File Share/ *Lbrule445* | |445 |

_**Tabla 1:** Números de puerto de las instancias de ASCS ABAP de SAP NetWeaver_

Luego, cree estos puntos de conexión de equilibrio de carga interno para los puertos de SCS de Java de SAP NetWeaver:

| Nombre de regla de equilibrio de carga/servicio | Números de puerto predeterminados | Puertos concretos para (instancia de SCS con el número de instancia 01) (ERS con 11) |
| --- | --- | --- |
| Enqueue Server/ *lbrule3201* |32<*NúmeroInstancia*> |3201 |
| Gateway Server/ *lbrule3301* |33<*NúmeroInstancia*> |3301 |
| Java Message Server/ *lbrule3900* |39<*NúmeroInstancia*> |3901 |
| Message Server HTTP/ *Lbrule8101* |81<*NúmeroInstancia*> |8101 |
| SAP Start Service SCS HTTP/ *Lbrule50113* |5<*NúmeroInstancia*>13 |50113 |
| SAP Start Service SCS HTTPS/ *Lbrule50114* |5<*NúmeroInstancia*>14 |50114 |
| Enqueue Replication/ *Lbrule50116* |5<*NúmeroInstancia*>16 |50116 |
| SAP Start Service ERS HTTP/ *Lbrule51113* |5<*NúmeroInstancia*>13 |51113 |
| SAP Start Service ERS HTTPS/ *Lbrule51114* |5<*NúmeroInstancia*>14 |51114 |
| Win RM/ *Lbrule5985* | |5985 |
| File Share/ *Lbrule445* | |445 |

_**Tabla 2:** Números de puerto de las instancias de SCS de Java de SAP NetWeaver_

![Figura 15: Reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure][sap-ha-guide-figure-3004]

_**Figura 15:** Reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure_

Establezca la dirección IP del equilibrador de carga **pr1-lb-dbms** en la dirección IP del nombre de host virtual de la instancia de DBMS.

### <a name="change-the-ascsscs-default-load-balancing-rules-for-the-azure-internal-load-balancer"></a><a name="fe0bd8b5-2b43-45e3-8295-80bee5415716"></a> Cambio de las reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure

Si desea usar otros números para las instancias de ASCS o SCS de SAP, debe actualizar los nombres y valores de sus puertos a partir de los predeterminados.

1. En Azure Portal, seleccione **<*SID*>-lb-ascs load balancer** > **Reglas de equilibrio de carga**.
2. Cambie estos valores para todas las reglas de equilibrio de carga que pertenezcan a la instancia de ASCS o SCS de SAP:

   * Nombre
   * Port
   * Puerto de back-end

   Por ejemplo, si desea cambiar el número de instancia de ASCS predeterminado de 00 a 31, debe hacer cambios en todos los puertos que aparecen en la tabla 1.

   A continuación, se muestra un ejemplo de una actualización para el puerto *lbrule3200*.

   ![Figura 16: Cambio de las reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure][sap-ha-guide-figure-3005]

   _**Figura 16:** Cambio de las reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure_

### <a name="add-windows-virtual-machines-to-the-domain"></a><a name="e69e9a34-4601-47a3-a41c-d2e11c626c0c"></a> Incorporación de máquinas virtuales de Windows al dominio

Después de asignar una dirección IP estática a las máquinas virtuales, agregue las máquinas virtuales al dominio.

![Figura 17: Incorporación de una máquina virtual a un dominio][sap-ha-guide-figure-3006]

_**Figura 17:** Incorporación de una máquina virtual a un dominio_

### <a name="add-registry-entries-on-both-cluster-nodes-of-the-sap-ascsscs-instance"></a><a name="661035b2-4d0f-4d31-86f8-dc0a50d78158"></a> Incorporación de entradas del Registro en ambos nodos del clúster usados para la instancia de ASCS/SCS de SAP

Azure Load Balancer tiene un equilibrador de carga interno que cierra las conexiones cuando están inactivas durante un período establecido (tiempo de espera de inactividad). Los proceso de trabajo de SAP en instancias de diálogo abren conexiones con el proceso de puesta en cola de SAP tan pronto como se deba enviar la primera solicitud para poner en cola y para quitar de la cola. Estas conexiones suelen mantenerse hasta que el proceso de trabajo o el de puesta en cola se reinicia. Sin embargo, si la conexión está inactiva durante un periodo, el equilibrador de carga interno de Azure cierra la conexión. Esto no supone un problema, porque el proceso de trabajo de SAP restablece la conexión con el proceso de puesta en cola si ya no existe. Estas actividades se documentan en los seguimientos de desarrollador de los procesos de SAP, pero crean una gran cantidad de contenido adicional en esos seguimientos. Se recomienda cambiar los parámetros de TCP/IP `KeepAliveTime` y `KeepAliveInterval` en ambos nodos del clúster. Combine estos cambios en los parámetros de TCP/IP con los parámetros de perfil de SAP, lo que se describe más adelante en este artículo.

Para agregar entradas de registro en los dos nodos de clúster de la instancia de ASCS/SCS de SAP, en primer lugar, agregue estas entradas del registro de Windows en ambos nodos de clúster de Windows para ASCS/SCS de SAP:

| Path | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| Nombre de la variable |`KeepAliveTime` |
| Tipo de variable |REG_DWORD (Decimal) |
| Value |120000 |
| Vínculo a la documentación |[https://technet.microsoft.com/library/cc957549.aspx](/previous-versions/windows/it-pro/windows-2000-server/cc957549(v=technet.10)) |

_**Tabla 3:** Cambio del primer parámetro de TCP/IP_

Luego, agregue estas entradas del Registro de Windows en los nodos de clúster de Windows para ASCS/SCS de SAP:

| Path | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| Nombre de la variable |`KeepAliveInterval` |
| Tipo de variable |REG_DWORD (Decimal) |
| Value |120000 |
| Vínculo a la documentación |[https://technet.microsoft.com/library/cc957548.aspx](/previous-versions/windows/it-pro/windows-2000-server/cc957548(v=technet.10)) |

_**Tabla 4:** Cambio del segundo parámetro de TCP/IP_

**Para aplicar los cambios, reinicie ambos nodos del clúster**.

### <a name="set-up-a-windows-server-failover-clustering-cluster-for-an-sap-ascsscs-instance"></a><a name="0d67f090-7928-43e0-8772-5ccbf8f59aab"></a> Configuración del clúster de Clústeres de conmutación por error de Windows para la instancia de ASCS/SCS de SAP

Para configurar el clúster de Clústeres de conmutación por error de Windows para la instancia de ASCS/SCS de SAP, hay que realizar estas tareas:

- Recopilación de nodos del clúster en la configuración de clúster
- Configuración de un testigo de recurso compartido de archivos de clúster

#### <a name="collect-the-cluster-nodes-in-a-cluster-configuration"></a><a name="5eecb071-c703-4ccc-ba6d-fe9c6ded9d79"></a> Recopilación de nodos del clúster en la configuración de clúster

1. En el Asistente para Agregar roles y características, agregue a ambos nodos del clúster agrupaciones en clústeres de conmutación por error.
2. Configure el clúster de conmutación por error mediante el Administrador de clústeres de conmutación por error. En el Administrador de clústeres de conmutación por error, seleccione **Crear clúster** y, luego, agregue solo el nombre del primer clúster, el nodo A. No agregue el segundo nodo aún, ya que lo agregará en un paso posterior.

   ![Figura 18: Incorporación del nombre de servidor o máquina virtual del primer nodo del clúster][sap-ha-guide-figure-3007]

   _**Figura 18:** Incorporación del nombre de servidor o máquina virtual del primer nodo del clúster_

3. Escriba el nombre de red (nombre de host virtual) del clúster.

   ![Figura 19: Escriba el nombre del clúster][sap-ha-guide-figure-3008]

   _**Figura 19:** Escriba el nombre del clúster_

4. Una vez creado el clúster, ejecute una prueba de validación del clúster.

   ![Figura 20: Ejecución de la comprobación de validación del clúster][sap-ha-guide-figure-3009]

   _**Figura 20:** Ejecución de la comprobación de validación del clúster_

   En este punto del proceso, puede omitir cualquier advertencia sobre los discos. Agregará un testigo de recurso de archivos y los discos compartidos de SIOS más tarde. En esta fase, no es necesario preocuparse por el cuórum.

   ![Figura 21: No se encuentra disco de cuórum][sap-ha-guide-figure-3010]

   _**Figura 21:** No se encuentra disco de cuórum_

   ![Figura 22: El recurso de clúster principal necesita una dirección IP nueva][sap-ha-guide-figure-3011]

   _**Figura 22:** El recurso de clúster principal necesita una dirección IP nueva_

5. Cambie la dirección IP del servicio de clúster principal. El clúster no se puede iniciar hasta que no cambie la dirección IP del servicio de clúster principal, porque la dirección IP del servidor apunta a uno de los nodos de la máquina virtual. Para ello, vaya a la página **Propiedad** del recurso de dirección IP del servicio de clúster principal.

   Por ejemplo, necesitamos asignar una dirección IP (**10.0.0.42** en el ejemplo) para el nombre de host virtual del clúster **pr1-ascs-vir**.

   ![Figura 23: En el cuadro de diálogo Propiedades, cambie la dirección IP][sap-ha-guide-figure-3012]

   _**Figura 23:** En el cuadro de diálogo **Propiedades**, cambie la dirección IP_

   ![Figura 24: Asignación de la dirección IP reservada para el clúster][sap-ha-guide-figure-3013]

   _**Figura 24:** Asignación de la dirección IP reservada para el clúster_

6. Escriba el nombre del host virtual del clúster en línea.

   ![Figura 25: El servicio principal del clúster está activo, en ejecución y tiene la dirección IP correcta][sap-ha-guide-figure-3014]

   _**Figura 25:** El servicio principal del clúster está activo, en ejecución y tiene la dirección IP correcta_

7. Agregue el segundo nodo del clúster.

   Ahora que el servicio de clúster principal está en funcionamiento, puede agregar el segundo nodo del clúster.

   ![Figura 26: Incorporación del segundo nodo del clúster][sap-ha-guide-figure-3015]

   _**Figura 26:** Incorporación del segundo nodo del clúster_

8. Escriba un nombre para el segundo host del nodo de clúster.

   ![Figura 27: Escriba el segundo nombre de host del nodo de clúster][sap-ha-guide-figure-3016]

   _**Figura 27:** Escriba el segundo nombre de host del nodo de clúster_

   > [!IMPORTANT]
   > Asegúrese de que la casilla **Agregar todo el almacenamiento apto al clúster** **NO** esté activada.  
   >
   >

   ![Figura 28: No active la casilla][sap-ha-guide-figure-3017]

   _**Figura 28:** **No** active la casilla_

   Puede pasar por alto las advertencias sobre el cuórum y los discos. Establecerá el cuórum y compartirá el disco más adelante, tal como se describe en [Instalación de SIOS DataKeeper Cluster Edition para un disco compartido de clúster de ASCS/SCS de SAP][sap-ha-guide-8.12.3].

   ![Figura 29: Omisión de las advertencias sobre el cuórum de disco][sap-ha-guide-figure-3018]

   _**Figura 29:** Omisión de las advertencias sobre el cuórum de disco_


#### <a name="configure-a-cluster-file-share-witness"></a><a name="e49a4529-50c9-4dcf-bde7-15a0c21d21ca"></a> Configuración de un testigo de recurso compartido de archivos de clúster

Para configurar un testigo de recurso compartido de archivos de clúster, hay que realizar las siguientes tareas:

- Creación de un recurso compartido de archivos
- Configuración del cuórum de testigo del recurso compartido de archivos en el Administrador de clústeres de conmutación por error

##### <a name="create-a-file-share"></a><a name="06260b30-d697-4c4d-b1c9-d22c0bd64855"></a> Creación de un recurso compartido de archivos

1. Seleccione un testigo de recurso de archivos en lugar de un disco de cuórum. SIOS DataKeeper admite esta opción.

   En los ejemplos de este artículo, el testigo de recurso compartido de archivos se encuentra en la instancia de Active Directory/el servidor DNS que se ejecuta en Azure. El testigo de recurso compartido de archivos se llama **domcontr-0**. Dado que habría configurado una conexión de VPN con Azure (a través de VPN de sitio a sitio o Azure ExpressRoute), su instancia de Active Directory/servidor DNS es local y no es apta para ejecutar un testigo de recurso compartido de archivos.

   > [!NOTE]
   > Si la instancia de Active Directory/el servidor DNS solo se ejecuta localmente, no configure el testigo de recurso compartido de archivos en la instancia de Active Directory/sistema operativo Windows de DNS que se ejecuta localmente. La latencia de red entre los nodos de clúster que se ejecutan en Azure y Active Directory/DNS local puede ser demasiado grande y provocar problemas de conectividad. Asegúrese de configurar el testigo de recurso compartido de archivos en una máquina virtual de Azure que se ejecuta cerca del nodo del clúster.  
   >
   >

   La unidad de cuórum necesita, como mínimo, 1024 MB de espacio disponible. Se recomienda 2048 MB de espacio disponible en la unidad de cuórum.

2. Agregue el objeto de nombre de clúster.

   ![Figura 30: Asignación de los permisos en el recurso compartido para el objeto de nombre de clúster][sap-ha-guide-figure-3019]

   _**Figura 30:** Asignación de los permisos en el recurso compartido para el objeto de nombre de clúster_

   Asegúrese de que los permisos incluyan la autoridad para cambiar datos en el recurso compartido para el objeto de nombre de clúster (**pr1-ascs-vir$** en el ejemplo).

3. Para agregar el objeto de nombre de clúster a la lista, seleccione **Agregar**. Cambie el filtro para buscar objetos de equipo, además de los que aparecen en la figura 31.

   ![Figura 31: Cambio de los tipos de objeto para incluir los equipos][sap-ha-guide-figure-3020]

   _**Figura 31:** Cambio de los tipos de objeto para incluir los equipos_

   ![Figura 32: Active la casilla Equipos][sap-ha-guide-figure-3021]

   _**Figura 32:** Active la casilla **Equipos**_

4. Escriba el objeto de nombre de clúster, como aparece en la figura 31. Como ya se creó el registro, puede cambiar los permisos, tal como aparece en la figura 30.

5. Seleccione la pestaña **Seguridad** del recurso compartido y, luego, establezca permisos más detallados para el objeto de nombre de clúster.

   ![Figura 33: Establecimiento de los atributos de seguridad para el objeto de nombre de clúster en el cuórum de recurso compartido de archivos][sap-ha-guide-figure-3022]

   _**Figura 33:** Establecimiento de los atributos de seguridad para el objeto de nombre de clúster en el cuórum de recurso compartido de archivos_

##### <a name="set-the-file-share-witness-quorum-in-failover-cluster-manager"></a><a name="4c08c387-78a0-46b1-9d27-b497b08cac3d"></a> Configuración del cuórum de testigo del recurso compartido de archivos en el Administrador de clústeres de conmutación por error

1. Abra el Asistente para la configuración de cuórum.

   ![Figura 34: Inicio del Asistente para configurar el cuórum de clúster][sap-ha-guide-figure-3023]

   _**Figura 34:** Inicio del Asistente para configurar el cuórum de clúster_

2. En la página **Seleccionar opción de configuración de cuórum**, seleccione **Seleccionar el testigo de cuórum**.

   ![Figura 35: Configuraciones de cuórum que puede elegir][sap-ha-guide-figure-3024]

   _**Figura 35:** Configuraciones de cuórum que puede elegir_

3. En la página **Seleccionar testigo de cuórum**, elija **Configurar un testigo de recurso compartido de archivos**.

   ![Figura 36: Selección del testigo de recurso compartido de archivos][sap-ha-guide-figure-3025]

   _**Figura 36:** Selección del testigo de recurso compartido de archivos_

4. Escriba la ruta de acceso UNC al recurso compartido de archivos (\\domcontr-0\FSW en el ejemplo). Seleccione **Siguiente** para ver una lista de los cambios que puede hacer.

   ![Figura 37: Definición de la ubicación del recurso compartido de archivos para el recurso compartido testigo][sap-ha-guide-figure-3026]

   _**Figura 37:** Definición de la ubicación del recurso compartido de archivos para el recurso compartido testigo_

5. Seleccione los cambios que desee y, luego, seleccione **Siguiente**. En este último paso, necesita volver a configurar correctamente el clúster, tal como aparece en la figura 38.  

   ![Figura 38: Confirmación de que volvió a configurar el clúster][sap-ha-guide-figure-3027]

   _**Figura 38:** Confirmación de que volvió a configurar el clúster_

Después de instalar correctamente el clúster de conmutación por error de Windows, se deben realizar cambios en algunos umbrales para adaptar la detección de conmutación por error a las condiciones de Azure. Los parámetros que se van a cambiar se documentan en este blog: https://blogs.msdn.microsoft.com/clustering/2012/11/21/tuning-failover-cluster-network-thresholds/. Suponiendo que las dos máquinas virtuales que compilación la configuración de clúster de Windows para ASCS/SCS están en la misma subred, los parámetros siguientes deben cambiarse a estos valores:
- SameSubNetDelay = 2
- SameSubNetThreshold = 15

Esta configuración se ha probado con los clientes y proporciona un buen compromiso para que, por un lado, sea lo suficientemente resistente. Por otro lado, dicha configuración proporciona una conmutación por error lo suficientemente rápida en condiciones de error real en caso de error de máquinas virtuales o nodos, o de software SAP. 

### <a name="install-sios-datakeeper-cluster-edition-for-the-sap-ascsscs-cluster-share-disk"></a><a name="5c8e5482-841e-45e1-a89d-a05c0907c868"></a> Instalación de SIOS DataKeeper Cluster Edition para un disco compartido de clúster de ASCS/SCS de SAP

Ahora tiene una configuración de Clústeres de conmutación por error de Windows Server activa en Azure. Pero, para instalar una instancia de ASCS/SCS de SAP, necesita un recurso de disco compartido. No puede crear los recursos de disco compartido que necesita en Azure. SIOS DataKeeper Cluster Edition es una solución de terceros que puede usar para crear recursos de disco compartido.

Para instalar SIOS DataKeeper Cluster Edition en un disco compartido de clúster de ASCS/SCS de SAP, siga estos pasos:

- Incorporación de .NET Framework 3.5
- Instalación de SIOS DataKeeper
- Configuración de SIOS DataKeeper

#### <a name="add-the-net-framework-35"></a><a name="1c2788c3-3648-4e82-9e0d-e058e475e2a3"></a> Incorporación de la característica .NET Framework 3.5
Microsoft .NET Framework 3.5 no está activo ni instalado automáticamente en Windows Server 2012 R2. Dado que SIOS DataKeeper requiere .NET Framework como en todos los nodos donde se instala DataKeeper, debe instalar .NET Framework 3.5 en el sistema operativo invitado de todas las máquinas virtuales del clúster.

Hay dos maneras de agregar .NET Framework 3.5:

- Use el Asistente para agregar roles y características en Windows, que se muestra en la figura 39.

  ![Figura 39: Instalación de .NET Framework 3.5 por medio del Asistente para agregar roles y características][sap-ha-guide-figure-3028]

  _**Figura 39:** Instalación de .NET Framework 3.5 por medio del Asistente para agregar roles y características_

  ![Figura 40: Barra de progreso de instalación cuando instala .NET Framework 3.5 por medio del Asistente para agregar roles y características][sap-ha-guide-figure-3029]

  _**Figura 40:** Barra de progreso de instalación cuando instala .NET Framework 3.5 por medio del Asistente para agregar roles y características_

- Utilice la herramienta de línea de comandos dism.exe. Para este tipo de instalación, necesita tener acceso al directorio SxS en los medios de instalación de Windows. En un símbolo del sistema con privilegios elevados, escriba:

  ```
  Dism /online /enable-feature /featurename:NetFx3 /All /Source:installation_media_drive:\sources\sxs /LimitAccess
  ```

#### <a name="install-sios-datakeeper"></a><a name="dd41d5a2-8083-415b-9878-839652812102"></a> Instalación de SIOS DataKeeper

Instale SIOS DataKeeper Cluster Edition en cada nodo del clúster. Con SIOS DataKeeper, para crear el almacenamiento compartido virtual, cree un reflejo sincronizado y, luego, simule el almacenamiento compartido de clúster.

Antes de instalar el software SIOS, cree el usuario de dominio **DataKeeperSvc**.

> [!NOTE]
> Agregue el usuario **DataKeeperSvc** al grupo de **administradores locales** en ambos nodos del clúster.
>
>

Para instalar SIOS DataKeeper, siga estos pasos:

1. Instale el software SIOS en ambos nodos del clúster.

   ![Instalador de SIOS][sap-ha-guide-figure-3030]

   ![Figura 41: Primera página de la instalación de SIOS DataKeeper][sap-ha-guide-figure-3031]

   _**Figura 41:** Primera página de la instalación de SIOS DataKeeper_

2. En el cuadro de diálogo que aparece en la figura 42, seleccione **Sí**.

   ![Figura 42: DataKeeper le informa que se deshabilitará un servicio][sap-ha-guide-figure-3032]

   _**Figura 42:** DataKeeper le informa que se deshabilitará un servicio_

3. En el cuadro de diálogo que aparece en la figura 43, se recomienda seleccionar **Cuenta de dominio o de servidor**.

   ![Figura 43: Selección del usuario para SIOS DataKeeper][sap-ha-guide-figure-3033]

   _**Figura 43:** Selección del usuario para SIOS DataKeeper_

4. Escriba el nombre de usuario y contraseñas de cuenta de dominio que creó para SIOS DataKeeper.

   ![Figura 44: Introducción del nombre de usuario y contraseña de dominio para la instalación de SIOS DataKeeper][sap-ha-guide-figure-3034]

   _**Figura 44:** Introducción del nombre de usuario y contraseña de dominio para la instalación de SIOS DataKeeper_

5. Instale la clave de licencia para la instancia de SIOS DataKeeper, tal como aparece en la figura 45.

   ![Figura 45: Especificación de la licencia de SIOS DataKeeper][sap-ha-guide-figure-3035]

   _**Figura 45:** Especificación de la licencia de SIOS DataKeeper_

6. Cuando se le solicite, reinicie la máquina virtual.

#### <a name="set-up-sios-datakeeper"></a><a name="d9c1fc8e-8710-4dff-bec2-1f535db7b006"></a> Configuración de SIOS DataKeeper

Después de instalar SIOS DataKeeper en ambos nodos, debe iniciar la configuración. El objetivo de la configuración es conseguir la replicación sincrónica de datos entre los discos adicionales asociados a cada una de las máquinas virtuales.

1. Inicie la herramienta de configuración y administración de DataKeeper y, luego, seleccione **Servidor de conexión**. (En la figura 46, esta opción aparece rodeada de un círculo de color rojo).

   ![Figura 46: Herramienta de configuración y administración de SIOS DataKeeper][sap-ha-guide-figure-3036]

   _**Figura 46:** Herramienta de configuración y administración de SIOS DataKeeper_

2. Escriba el nombre o la dirección TCP/IP del primer nodo al que se debe conectar la herramienta de configuración y administración y, en un segundo paso, el segundo nodo.

   ![Figura 47: Inserte el nombre o la dirección TCP/IP del primer nodo al que se debe conectar la herramienta de configuración y administración y, en un segundo paso, el segundo nodo][sap-ha-guide-figure-3037]

   _**Figura 47:** Inserte el nombre o la dirección TCP/IP del primer nodo al que se debe conectar la herramienta de configuración y administración y, en un segundo paso, el segundo nodo_

3. Cree el trabajo de replicación entre los dos nodos.

   ![Figura 48: Creación de un trabajo de replicación][sap-ha-guide-figure-3038]

   _**Figura 48:** Creación de un trabajo de replicación_

   Un asistente le guía por el proceso de crear un trabajo de replicación.
4. Defina el nombre, la dirección TCP/IP y el volumen de disco del nodo de origen.

   ![Figura 49: Definición del nombre del trabajo de replicación][sap-ha-guide-figure-3039]

   _**Figura 49:** Definición del nombre del trabajo de replicación_

   ![Figura 50: Definición de los datos básicos para el nodo que debe ser el nodo de origen actual][sap-ha-guide-figure-3040]

   _**Figura 50:** Definición de los datos básicos para el nodo que debe ser el nodo de origen actual_

5. Defina el nombre, la dirección TCP/IP y el volumen de disco del nodo de destino.

   ![Figura 51: Definición de los datos básicos para el nodo que debe ser el nodo de destino actual][sap-ha-guide-figure-3041]

   _**Figura 51:** Definición de los datos básicos para el nodo que debe ser el nodo de destino actual_

6. Defina los algoritmos de compresión. En el ejemplo, se recomienda comprimir el flujo de replicación. Especialmente en situaciones de resincronización, la compresión del flujo de replicación reduce considerablemente el tiempo que se tarda en resincronizar. Tenga en cuenta que la compresión usa los recursos de CPU y RAM de una máquina virtual. A medida que aumenta la tasa de compresión, también aumenta el volumen de los recursos de CPU usados. También puede ajustar esta configuración más adelante.

7. Otra configuración que debe comprobar es si la replicación se ejecuta de forma sincrónica o asincrónica. *Cuando proteja las configuraciones de ASCS/SCS de SAP, debe usar la replicación sincrónica*.  

   ![Figura 52: Definición de los detalles de la replicación][sap-ha-guide-figure-3042]

   _**Figura 52:** Definición de los detalles de la replicación_

8. Defina si el volumen que el trabajo de replicación replica se debe representar en una configuración de clúster de Clústeres de conmutación por error de Windows Server como disco compartido. Para la configuración de ASCS/SCS de SAP, seleccione **Sí** de manera que el clúster de Windows vea el volumen replicado como un disco compartido que puede usar como volumen de clúster.

   ![Figura 53: Selección de Sí para establecer el volumen replicado como volumen de clúster][sap-ha-guide-figure-3043]

   _**Figura 53:** Selección de **Sí** para establecer el volumen replicado como volumen de clúster_

   Una vez creado el volumen, la herramienta de configuración y administración de DataKeeper muestra que el trabajo de replicación está activo.

   ![Figura 54: El reflejo sincrónico de DataKeeper para el disco compartido de ASCS/SCS de SAP está activo][sap-ha-guide-figure-3044]

   _**Figura 54:** El reflejo sincrónico de DataKeeper para el disco compartido de ASCS/SCS de SAP está activo_

   Ahora, el Administrador de clústeres de conmutación por error muestra el disco como un disco de DataKeeper, tal como aparece en la figura 55.

   ![Figura 55: El Administrador de clústeres de conmutación por error muestra el disco que DataKeeper replicó][sap-ha-guide-figure-3045]

   _**Figura 55:** El Administrador de clústeres de conmutación por error muestra el disco que DataKeeper replicó_

## <a name="install-the-sap-netweaver-system"></a><a name="a06f0b49-8a7a-42bf-8b0d-c12026c5746b"></a> Instalación del sistema SAP NetWeaver

No se describirá la configuración de DBMS, porque las configuraciones varían en función del sistema de DBMS que usa. Sin embargo, se da por supuesto que las inquietudes respecto de la alta disponibilidad con DBMS se abordan con las funcionalidades que admiten los diversos proveedores de DBMS para Azure. Por ejemplo, AlwaysOn o creación de reflejo de la base de datos para SQL Server y Oracle Data Guard para Oracle. En el escenario de este artículo, no se agregó más protección a DBMS.

No hay ninguna consideración especial cuando distintos servicios de DBMS interactúan con esta variante de configuración de ASCS/SCS de SAP en clúster en Azure.

> [!NOTE]
> Los procedimientos de instalación de sistemas ABAP de SAP NetWeaver, sistemas Java y sistemas ABAP+Java son casi idénticos. La diferencia más significativa es que un sistema ABAP de SAP tiene una instancia de ASCS. El sistema Java de SAP tiene una instancia de SCS. El sistema ABAP+Java de SAP tiene una instancia de ASCS y una instancia de SCS en ejecución en el mismo grupo de clústeres de conmutación por error de Microsoft. Cualquier diferencia de instalación de cada pila de instalación de SAP NetWeaver se menciona explícitamente. Puede asumir que todos los demás componentes son iguales.  
>
>

### <a name="install-sap-with-a-high-availability-ascsscs-instance"></a><a name="31c6bd4f-51df-4057-9fdf-3fcbc619c170"></a> Instalación de SAP con una instancia de ASCS/SCS de alta disponibilidad

> [!IMPORTANT]
> Asegúrese de no colocar el archivo de paginación en los volúmenes reflejados de DataKeeper. DataKeeper no admite los volúmenes reflejados. Puede dejar el archivo de paginación en la unidad temporal D de una máquina virtual de Azure, que es la opción predeterminada. Si todavía no se encuentra ahí, mueva el archivo de paginación de Windows a la unidad D de la máquina virtual de Azure.
>
>

Para instalar SAP con una instancia de ASCS/SCS de alta disponibilidad, siga estos pasos:

- Creación de un nombre de host virtual para la instancia de ASCS/SCS de SAP en clúster
- Instalación del primer nodo de clúster de SAP
- Modificación del perfil SAP de la instancia de ASCS/SCS
- Incorporación de un puerto de sondeo
- Apertura del puerto de sondeo de Firewall de Windows

#### <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance"></a><a name="a97ad604-9094-44fe-a364-f89cb39bf097"></a> Creación de un nombre de host virtual para la instancia de ASCS/SCS de SAP en clúster

1. En el Administrador de DNS de Windows, cree una entrada de DNS para el nombre de host virtual de la instancia de ASCS/SCS.

   > [!IMPORTANT]
   > La dirección IP que asigna al nombre de host virtual de la instancia de ASCS/SCS debe ser la misma dirección IP que asignó a Azure Load Balancer ( **<*SID*>-lb-ascs**).  
   >
   >

   La dirección IP del nombre de host de ASCS/SCS de SAP virtual (**pr1-ascs-sap**) es la misma dirección IP de Azure Load Balancer (**pr1-lb-ascs**).

   ![Figura 56: Definición de la entrada DNS del nombre virtual del clúster de ASCS/SCS de SAP y la dirección TCP/IP][sap-ha-guide-figure-3046]

   _**Figura 56:** Definición de la entrada DNS del nombre virtual del clúster de ASCS/SCS de SAP y la dirección TCP/IP_

2. Para definir la dirección IP asignada al nombre de host virtual, seleccione **Administrador de DNS** > **Dominio**.

   ![Figura 57: Nuevo nombre virtual y dirección TCP/IP para la configuración del clúster de ASCS/SCS de SAP][sap-ha-guide-figure-3047]

   _**Figura 57:** Nuevo nombre virtual y dirección TCP/IP para la configuración del clúster de ASCS/SCS de SAP_

#### <a name="install-the-sap-first-cluster-node"></a><a name="eb5af918-b42f-4803-bb50-eff41f84b0b0"></a> Instalación del primer nodo de clúster de SAP

1. Ejecute la opción de primer nodo del clúster en el nodo de clúster A, por ejemplo, en el host **pr1-ascs-0**.
2. Para mantener los puertos predeterminados para el equilibrador de carga interno de Azure, seleccione:

   * **Sistema ABAP**: número de instancia de **ASCS** **00**
   * **Sistema Java**: número de instancia de **SCS** **01**
   * **Sistema ABAP+Java**: número de instancia de **ASCS** **00** y número de instancia de **SCS** **01**

   Para usar números de instancia diferentes de 00 para la instancia de ASCS de ABAP y 01 para la instancia de SCS de Java, primero debe cambiar las reglas predeterminadas de equilibrio de carga del equilibrador de carga interno de Azure, tal como se describe en [Cambio de las reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure][sap-ha-guide-8.9].

Luego, realice unos pasos que no se describen en la documentación de instalación usual de SAP.

> [!NOTE]
> La documentación de instalación de SAP describe cómo instalar el primer nodo de clúster de ASCS/SCS.
>
>

#### <a name="modify-the-sap-profile-of-the-ascsscs-instance"></a><a name="e4caaab2-e90f-4f2c-bc84-2cd2e12a9556"></a> Modificación del perfil SAP de la instancia de ASCS/SCS

Debe agregar un nuevo parámetro de perfil. El parámetro de perfil evita el cierre de las conexiones entre los procesos de trabajo de SAP y el servidor de puesta en cola cuando lleven inactivas demasiado tiempo. Mencionamos el escenario del problema en [Incorporación de entradas del Registro en ambos nodos del clúster usados para la instancia de ASCS/SCS de SAP][sap-ha-guide-8.11]. En esa sección, también se presentan 2 cambios para algunos parámetros básicos de la conexión TCP/IP. En el segundo paso, es necesario configurar el servidor de puesta en cola para enviar una señal `keep_alive` de forma que las conexiones no alcancen el umbral de inactividad del equilibrador de carga interno de Azure.

Para modificar el perfil SAP de la instancia de ASCS/SCS, siga estos pasos:

1. Agregue este parámetro de perfil al perfil de la instancia de ASCS/SCS de SAP:

   ```
   enque/encni/set_so_keepalive = true
   ```
   En este ejemplo, la ruta de acceso es:

   `<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_ASCS00_pr1-ascs-sap`

   Por ejemplo, al perfil de la instancia de SCS de SAP y la ruta de acceso correspondiente:

   `<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_SCS01_pr1-ascs-sap`

2. Para aplicar los cambios, reinicie la instancia de ASCS/SCS de SAP.

#### <a name="add-a-probe-port"></a><a name="10822f4f-32e7-4871-b63a-9b86c76ce761"></a> Incorporación de un puerto de sondeo

Use la funcionalidad de sondeo del equilibrador de carga interno para que toda la configuración del clúster funcione con Azure Load Balancer. Habitualmente, el equilibrador de carga interno de Azure distribuye la carga de trabajo entrante de manera equitativa entre las máquinas virtuales que participan. Sin embargo, esto no funcionará en algunas configuraciones de clúster porque solo hay una instancia activa. La otra instancia es pasiva y no puede aceptar nada de la carga de trabajo. Una funcionalidad de sondeo resulta útil cuando el equilibrador de carga interno de Azure asigna trabajo solo a una instancia activa. Con la funcionalidad de sondeo, el equilibrador de carga interno puede detectar cuáles son las instancias activas y, luego, orientarse solo a la instancia con la carga de trabajo.

Para agregar un puerto de sondeo, siga estos pasos:

1. Primero, compruebe la configuración de **ProbePort** actual con este comando de PowerShell. Ejecútelo dentro de una de las máquinas virtuales de la configuración del clúster.

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPNetworkIPClusterName = "SAP $SAPSID IP"
   Get-ClusterResource $SAPNetworkIPClusterName | Get-ClusterParameter
   ```

2. Defina un puerto de sondeo. El número de puerto de sondeo predeterminado es **0**. En el ejemplo, se usa el puerto de sondeo **62000**.

   ![Figura 58: De manera predeterminada, el puerto de sondeo de configuración de clúster es 0][sap-ha-guide-figure-3048]

   _**Figura 58:** El puerto de sondeo de configuración de clúster es 0_

   El número de puerto está definido en las plantillas de Azure Resource Manager para SAP. Puede asignar el número de puerto en PowerShell.

   Para establecer un nuevo valor de ProbePort del recurso de clúster **SAP <*SID*> IP**, ejecute el siguiente script de PowerShell. Actualice las variables de PowerShell de su entorno. Después de ejecutar el script, se le pedirá que reinicie el grupo de clústeres de SAP para activar los cambios.

   ```powershell
   $SAPSID = "PR1"      # SAP <SID>
   $ProbePort = 62000   # ProbePort of the Azure Internal Load Balancer

   Clear-Host
   $SAPClusterRoleName = "SAP $SAPSID"
   $SAPIPresourceName = "SAP $SAPSID IP"
   $SAPIPResourceClusterParameters =  Get-ClusterResource $SAPIPresourceName | Get-ClusterParameter
   $IPAddress = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Address" }).Value
   $NetworkName = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Network" }).Value
   $SubnetMask = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "SubnetMask" }).Value
   $OverrideAddressMatch = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "OverrideAddressMatch" }).Value
   $EnableDhcp = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "EnableDhcp" }).Value
   $OldProbePort = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "ProbePort" }).Value

   $var = Get-ClusterResource | Where-Object {  $_.name -eq $SAPIPresourceName  }

   Write-Host "Current configuration parameters for SAP IP cluster resource '$SAPIPresourceName' are:" -ForegroundColor Cyan
   Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter

   Write-Host
   Write-Host "Current probe port property of the SAP cluster resource '$SAPIPresourceName' is '$OldProbePort'." -ForegroundColor Cyan
   Write-Host
   Write-Host "Setting the new probe port property of the SAP cluster resource '$SAPIPresourceName' to '$ProbePort' ..." -ForegroundColor Cyan
   Write-Host

   $var | Set-ClusterParameter -Multiple @{"Address"=$IPAddress;"ProbePort"=$ProbePort;"Subnetmask"=$SubnetMask;"Network"=$NetworkName;"OverrideAddressMatch"=$OverrideAddressMatch;"EnableDhcp"=$EnableDhcp}

   Write-Host

   $ActivateChanges = Read-Host "Do you want to take restart SAP cluster role '$SAPClusterRoleName', to activate the changes (yes/no)?"

   if($ActivateChanges -eq "yes"){
   Write-Host
   Write-Host "Activating changes..." -ForegroundColor Cyan

   Write-Host
   write-host "Taking SAP cluster IP resource '$SAPIPresourceName' offline ..." -ForegroundColor Cyan
   Stop-ClusterResource -Name $SAPIPresourceName
   sleep 5

   Write-Host "Starting SAP cluster role '$SAPClusterRoleName' ..." -ForegroundColor Cyan
   Start-ClusterGroup -Name $SAPClusterRoleName

   Write-Host "New ProbePort parameter is active." -ForegroundColor Green
   Write-Host

   Write-Host "New configuration parameters for SAP IP cluster resource '$SAPIPresourceName':" -ForegroundColor Cyan
   Write-Host
   Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter
   }else
   {
   Write-Host "Changes are not activated."
   }
   ```

   Después de conectar el rol de clúster **SAP <*SID*>** , compruebe que **ProbePort** esté establecido en el nuevo valor.

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPNetworkIPClusterName = "SAP $SAPSID IP"
   Get-ClusterResource $SAPNetworkIPClusterName | Get-ClusterParameter

   ```

   ![Figura 59: Sondeo del puerto de clúster después de establecer el valor nuevo][sap-ha-guide-figure-3049]

   _**Figura 59:** Sondeo del puerto de clúster después de establecer el valor nuevo_

#### <a name="open-the-windows-firewall-probe-port"></a><a name="4498c707-86c0-4cde-9c69-058a7ab8c3ac"></a> Apertura del puerto de sondeo de Firewall de Windows

Debe abrir el puerto de sondeo de firewall de Windows en ambos nodos del clúster. El siguiente script permite abrir un puerto de sondeo de firewall de Windows. Actualice las variables de PowerShell de su entorno.

  ```powershell
  $ProbePort = 62000   # ProbePort of the Azure Internal Load Balancer

  New-NetFirewallRule -Name AzureProbePort -DisplayName "Rule for Azure Probe Port" -Direction Inbound -Action Allow -Protocol TCP -LocalPort $ProbePort
  ```

El valor **ProbePort** está establecido en **62000**. Ahora puede tener acceso al recurso compartido de archivos **\\\ascsha-clsap\sapmnt** desde otros hosts, como **ascsha-dbas**.

### <a name="install-the-database-instance"></a><a name="85d78414-b21d-4097-92b6-34d8bcb724b7"></a> Instalación de la instancia de base de datos

Para instalar la instancia de base de datos, siga el proceso que se describe en la documentación de instalación de SAP.

### <a name="install-the-second-cluster-node"></a><a name="8a276e16-f507-4071-b829-cdc0a4d36748"></a> Instalación del segundo nodo del clúster

Para instalar el segundo clúster, siga los pasos que aparecen en la guía de instalación de SAP.

### <a name="change-the-start-type-of-the-sap-ers-windows-service-instance"></a><a name="094bc895-31d4-4471-91cc-1513b64e406a"></a> Cambio del tipo de inicio del servicio de Windows para la instancia de ERS de SAP

Cambie el tipo de inicio del servicio o servicios de Windows de ERS de SAP a **Automático (inicio retrasado)** en ambos nodos del clúster.

![Figura 60: Cambio del tipo de servicio de la instancia de ERS de SAP a automático retrasado][sap-ha-guide-figure-3050]

_**Figura 60:** Cambio del tipo de servicio de la instancia de ERS de SAP a automático retrasado_

### <a name="install-the-sap-primary-application-server"></a><a name="2477e58f-c5a7-4a5d-9ae3-7b91022cafb5"></a> Instalación del servidor de aplicaciones principal de SAP

Instale la instancia del servidor de aplicaciones principal (PAS) <*SID*>-di-0 en la máquina virtual que designó para hospedar el PAS. No hay ninguna dependencia de Azure ni aspectos específicos de DataKeeper.

### <a name="install-the-sap-additional-application-server"></a><a name="0ba4a6c1-cc37-4bcf-a8dc-025de4263772"></a> Instalación del servidor de aplicaciones adicional de SAP

Instale un servidor de aplicaciones adicional (AAS) de SAP en todas las máquinas virtuales que designó para hospedar una instancia de servidores de aplicaciones de SAP. Por ejemplo, en <*SID*>-di-1 to <*SID*>-di-&lt;n&gt;.

> [!NOTE]
> Esta acción finaliza la instalación de un sistema SAP NetWeaver de alta disponibilidad. Después, continúe con las pruebas de conmutación por error.
>


## <a name="test-the-sap-ascsscs-instance-failover-and-sios-replication"></a><a name="18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9"></a> Prueba de la conmutación por error de la instancia de ASCS/SCS de SAP y de la replicación de SIOS
Es fácil probar y supervisar la conmutación por error de la instancia de ASCS/SCS de SAP y la replicación de disco de SIOS mediante el uso del Administrador de clústeres de conmutación por error, la interfaz de usuario de SIOS DataKeeper y la herramienta de configuración.

### <a name="sap-ascsscs-instance-is-running-on-cluster-node-a"></a><a name="65fdef0f-9f94-41f9-b314-ea45bbfea445"></a> La instancia de ASCS/SCS de SAP se ejecuta en el nodo de clúster A

El grupo de clústeres **SAP PR1** se ejecuta en el nodo de clúster A. Por ejemplo, en **pr1-ascs-0**. Asigne la unidad de disco compartido S, que forma parte del grupo de clústeres **SAP PR1** y que la instancia de ASCS/SCS usa, al nodo de clúster A.

![Figura 61: Administrador de clústeres de conmutación por error: El grupo de clústeres <SID> de SAP se ejecuta en el nodo del clúster A][sap-ha-guide-figure-5000]

_**Figura 61:** Administrador de clústeres de conmutación por error: El grupo de clústeres <*SID*> de SAP se ejecuta en el nodo del clúster A_

En la interfaz de usuario de SIOS DataKeeper y la herramienta de configuración, puede ver que los datos del disco compartido se replican sincrónicamente desde la unidad de volumen de origen S en el nodo de clúster A. Por ejemplo, de **pr1-ascs-0 [10.0.0.40]** a **pr1-ascs-1 [10.0.0.41]** .

![Figura 62: En SIOS DataKeeper, replique el volumen local desde el nodo de clústeres A al nodo de clústeres B][sap-ha-guide-figure-5001]

_**Figura 62:** En SIOS DataKeeper, replique el volumen local desde el nodo de clústeres A al nodo de clústeres B_

### <a name="failover-from-node-a-to-node-b"></a><a name="5e959fa9-8fcd-49e5-a12c-37f6ba07b916"></a> Conmutación por error del nodo A al nodo B

1. Elija una de estas opciones para iniciar una conmutación por error del grupo de clústeres <*SID*> de SAP desde el nodo del clúster A al nodo del clúster B:
   - Uso del Administrador de clústeres de conmutación por error  
   - Uso de PowerShell del clúster de conmutación por error

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPClusterGroup = "SAP $SAPSID"
   Move-ClusterGroup -Name $SAPClusterGroup

   ```
2. Reinicie el nodo del clúster A dentro del sistema operativo invitado Windows (inicia una conmutación por error automática del grupo de clústeres <*SID*> de SAP desde el nodo A al nodo B).  
3. Reinicie el nodo del clúster A desde Azure Portal (inicia una conmutación por error automática del grupo de clústeres <*SID*> de SAP desde el nodo A al nodo B).  
4. Reinicie el nodo del clúster A con Azure PowerShell (inicia una conmutación por error automática del grupo de clústeres de SAP <*SID*> desde el nodo A al nodo B).

   Después de la conmutación por error, el grupo de clústeres <*SID*> de SAP se ejecuta en el nodo del clúster B. Por ejemplo, en **pr1-ascs-1**.

   ![Figura 63: En el Administrador de clústeres de conmutación por error, el grupo de clústeres <SID> de SAP se ejecuta en el nodo del clúster B][sap-ha-guide-figure-5002]

   _**Figura 63**: En el Administrador de clústeres de conmutación por error, el grupo de clústeres <*SID*> de SAP se ejecuta en el nodo del clúster B_

   Ahora, el disco compartido ahora está montado en el nodo del clúster B. SIOS DataKeeper replica datos desde la unidad de volumen de origen S en el nodo del clúster B a la unidad de volumen de destino S en el nodo del clúster A. Por ejemplo, de **pr1-ascs-1 [10.0.0.41]** a **pr1-ascs-0 [10.0.0.40]** .

   ![Figura 64: SIOS DataKeeper replica el volumen local desde el nodo del clúster B al nodo del clúster A][sap-ha-guide-figure-5003]

   _**Figura 64:** SIOS DataKeeper replica el volumen local desde el nodo del clúster B al nodo del clúster A_
