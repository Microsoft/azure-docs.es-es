---
title: Instalación de alta disponibilidad para SAP NetWeaver en un clúster de conmutación por error de Windows y un disco compartido para una instancia de ASCS/SCS de SAP en Azure | Microsoft Docs
description: Descubra cómo realizar una instalación de alta disponibilidad para SAP NetWeaver en un clúster de conmutación por error de Windows y un disco compartido para una instancia de ASCS/SCS de SAP.
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 6209bcb3-5b20-4845-aa10-1475c576659f
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/04/2020
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 97da7428090935daf95ae28a54b8ff10bca2e546
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87760913"
---
# <a name="install-sap-netweaver-ha-on-a-windows-failover-cluster-and-shared-disk-for-an-sap-ascsscs-instance-in-azure"></a>Instalación de alta disponibilidad para SAP NetWeaver en un clúster de conmutación por error de Windows y un disco compartido para una instancia de ASCS/SCS de SAP en Azure

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

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
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-ha-guide-8.9]:high-availability-guide.md#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-ha-guide-8.11]:high-availability-guide.md#661035b2-4d0f-4d31-86f8-dc0a50d78158
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md



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

Este artículo describe cómo instalar y configurar un sistema SAP de alta disponibilidad en Azure mediante el uso de un clúster de conmutación por error de Windows Server y un disco compartido de clúster para agrupar en clústeres una instancia de SAP ASCS/SCS.

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar la instalación, consulte estos documentos:

* [Guía de arquitectura: Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un disco compartido de clúster][sap-high-availability-guide-wsfc-shared-disk]

* [Preparación de la infraestructura de Azure para alta disponibilidad de SAP con un clúster de conmutación por error de Windows y un disco compartido para ASCS/SCS de SAP][sap-high-availability-infrastructure-wsfc-shared-disk]

En este artículo no se describe la configuración de DBMS porque las opciones varían en función del sistema DBMS que use. Se da por supuesto que las cuestiones relacionadas con la alta disponibilidad con DBMS se solucionan con las funcionalidades que admiten los diversos proveedores de DBMS para Azure. Por ejemplo, AlwaysOn o creación de reflejo de la base de datos para SQL Server y Oracle Data Guard para bases de datos de Oracle. En el escenario de este artículo, no se añade más protección a DBMS.

No hay ninguna consideración especial cuando distintos servicios de DBMS interactúan con una configuración de ASCS/SCS de SAP en clúster en Azure.

> [!NOTE]
> Los procedimientos de instalación de sistemas ABAP de SAP NetWeaver, sistemas Java y sistemas ABAP+Java son casi idénticos. La diferencia más significativa es que un sistema ABAP de SAP tiene una instancia de ASCS. El sistema Java de SAP tiene una instancia de SCS. El sistema ABAP+Java de SAP tiene una instancia de ASCS y una instancia de SCS en ejecución en el mismo grupo de clústeres de conmutación por error de Microsoft. Cualquier diferencia de instalación de cada pila de instalación de SAP NetWeaver se menciona explícitamente. Puede asumir que todos los demás componentes son iguales.  
>
>

## <a name="install-sap-with-a-high-availability-ascsscs-instance"></a><a name="31c6bd4f-51df-4057-9fdf-3fcbc619c170"></a>Instalación de SAP con una instancia de ASCS/SCS de alta disponibilidad

> [!IMPORTANT]
> Asegúrese de no colocar el archivo de paginación en los volúmenes reflejados de SIOS DataKeeper. DataKeeper no admite los volúmenes reflejados. Puede dejar el archivo de paginación en la unidad temporal D de una máquina virtual de Azure, que es la opción predeterminada. Si todavía no se encuentra ahí, mueva el archivo de paginación de Windows a la unidad D de la máquina virtual de Azure.
>
>

Para instalar SAP con una instancia de ASCS/SCS de alta disponibilidad, siga estos pasos:

* Cree un nombre de host virtual para la instancia de ASCS/SCS de SAP en clúster.
* Instale el primer nodo de clúster de SAP.
* Modifique el perfil SAP de la instancia de ASCS/SCS.
* Añada un puerto de sondeo.
* Abra el puerto de sondeo de firewall de Windows.

### <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance"></a><a name="a97ad604-9094-44fe-a364-f89cb39bf097"></a>Creación de un nombre de host virtual para la instancia de ASCS/SCS de SAP en clúster

1. En el Administrador de DNS de Windows, cree una entrada de DNS para el nombre de host virtual de la instancia de ASCS/SCS.

   > [!IMPORTANT]
   > La dirección IP que asigna al nombre de host virtual de la instancia de ASCS/SCS debe ser la misma dirección IP que asignó a Azure Load Balancer (\<SID\>-lb-ascs).  
   >
   >

   La dirección IP del nombre de host de ASCS/SCS de SAP virtual (pr1-ascs-sap) es la misma dirección IP de Azure Load Balancer (pr1-lb-ascs).

   ![Figura 1: Definición de la entrada DNS del nombre virtual del clúster de ASCS/SCS de SAP y la dirección TCP/IP][sap-ha-guide-figure-3046]

   _**Ilustración 1:** Definición de la entrada DNS del nombre virtual del clúster de ASCS/SCS de SAP y la dirección TCP/IP_

2. Para definir la dirección IP asignada al nombre de host virtual, seleccione **Administrador de DNS** > **Dominio**.

   ![Ilustración 2: Nuevo nombre virtual y dirección TCP/IP para la configuración del clúster de ASCS/SCS de SAP][sap-ha-guide-figure-3047]

   _**Ilustración 2:** Nuevo nombre virtual y dirección TCP/IP para la configuración del clúster de ASCS/SCS de SAP_

### <a name="install-the-sap-first-cluster-node"></a><a name="eb5af918-b42f-4803-bb50-eff41f84b0b0"></a> Instalación del primer nodo de clúster de SAP

1. Ejecute la opción de primer nodo del clúster en el nodo de clúster A, por ejemplo, en el host pr1-ascs-0*.
2. Para mantener los puertos predeterminados para el equilibrador de carga interno de Azure, seleccione:

   * **Sistema ABAP**: número de instancia de **ASCS** **00**
   * **Sistema Java**: número de instancia de **SCS** **01**
   * **Sistema ABAP+Java**: número de instancia de **ASCS** **00** y número de instancia de **SCS** **01**

   Para utilizar números de instancia distintos de 00 para la instancia de ABAP ASCS y 01 para la instancia de SCS de Java, en primer lugar, cambie las reglas predeterminadas de equilibrio de carga para el equilibrador de carga interno de Azure. Para obtener más información, vea [Cambio de las reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure][sap-ha-guide-8.9].

Luego, realice unos pasos que no se describen en la documentación de instalación usual de SAP.

> [!NOTE]
> La documentación de instalación de SAP describe cómo instalar el primer nodo de clúster de ASCS/SCS.
>
>

### <a name="modify-the-sap-profile-of-the-ascsscs-instance"></a><a name="e4caaab2-e90f-4f2c-bc84-2cd2e12a9556"></a> Modificación del perfil SAP de la instancia de ASCS/SCS

En primer lugar, añada un nuevo parámetro de perfil. El parámetro de perfil evita el cierre de las conexiones entre los procesos de trabajo de SAP y el servidor de puesta en cola cuando lleven inactivas demasiado tiempo. El escenario del problema se menciona en [Incorporación de entradas del Registro en ambos nodos del clúster usados para la instancia de ASCS/SCS de SAP][sap-ha-guide-8.11]. En esa sección, también se presentan dos cambios para algunos parámetros básicos de la conexión TCP/IP. En el segundo paso, es necesario configurar el servidor de puesta en cola para enviar una señal `keep_alive` de forma que las conexiones no alcancen el umbral de inactividad del equilibrador de carga interno de Azure.

Para modificar el perfil SAP de la instancia de ASCS/SCS, siga estos pasos:

1. Agregue este parámetro de perfil al perfil de la instancia de ASCS/SCS de SAP si usa ENSA1.

   ```
   enque/encni/set_so_keepalive = true
   ```
   En este ejemplo, la ruta de acceso es:

   `<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_ASCS00_pr1-ascs-sap`

   Por ejemplo, al perfil de la instancia de SCS de SAP y la ruta de acceso correspondiente:

   `<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_SCS01_pr1-ascs-sap`
   
   En el caso de ENSA1 y ENSA2, asegúrese de que los parámetros del sistema operativo `keepalive` se establecen tal y como se describe en la nota de SAP [1410736](https://launchpad.support.sap.com/#/notes/1410736).   

2. Para aplicar los cambios, reinicie la instancia de ASCS/SCS de SAP.

### <a name="add-a-probe-port"></a><a name="10822f4f-32e7-4871-b63a-9b86c76ce761"></a> Incorporación de un puerto de sondeo

Use la funcionalidad de sondeo del equilibrador de carga interno para que toda la configuración del clúster funcione con Azure Load Balancer. Habitualmente, el equilibrador de carga interno de Azure distribuye la carga de trabajo entrante de manera equitativa entre las máquinas virtuales que participan.

 Sin embargo, esto no funcionará en algunas configuraciones de clúster porque solo hay una instancia activa. La otra instancia es pasiva y no puede aceptar nada de la carga de trabajo. Una funcionalidad de sondeo resulta útil cuando el equilibrador de carga interno de Azure asigna trabajo solo a una instancia activa. Con la funcionalidad de sondeo, el equilibrador de carga interno puede detectar cuáles son las instancias activas y, luego, orientarse solo a la instancia con la carga de trabajo.

Para agregar un puerto de sondeo, siga estos pasos:

1. Compruebe el valor de **ProbePort** actual con este comando de PowerShell:

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPNetworkIPClusterName = "SAP $SAPSID IP"
   Get-ClusterResource $SAPNetworkIPClusterName | Get-ClusterParameter
   ```

   Ejecute el comando dentro de una de las máquinas virtuales de la configuración del clúster.

2. Defina un puerto de sondeo. El número de puerto de sondeo predeterminado es 0. En el ejemplo, se usa el puerto de sondeo 62000.

   ![Ilustración 3: De manera predeterminada, el puerto de sondeo de configuración de clúster es 0][sap-ha-guide-figure-3048]

   _**Ilustración 3:** El puerto de sondeo de configuración de clúster es 0_

   El número de puerto está definido en las plantillas de Azure Resource Manager para SAP. Puede asignar el número de puerto en PowerShell.

   Para establecer un nuevo valor de ProbePort del recurso de clúster de IP de \<SID\> de SAP, ejecute el siguiente script de PowerShell para actualizar las variables de PowerShell para el entorno:

   ```powershell
   $SAPSID = "PR1"      # SAP <SID>
   $ProbePort = 62000   # ProbePort of the Azure internal load balancer

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

   Después de conectar el rol de clúster \<SID\> de SAP, compruebe que **ProbePort** esté establecido en el nuevo valor.

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPNetworkIPClusterName = "SAP $SAPSID IP"
   Get-ClusterResource $SAPNetworkIPClusterName | Get-ClusterParameter

   ```
   Después de ejecutar el script, se le pedirá que reinicie el grupo de clústeres de SAP para activar los cambios.

   ![Ilustración 4: Sondeo del puerto de clúster después de establecer el valor nuevo][sap-ha-guide-figure-3049]

   _**Ilustración 4:** Sondeo del puerto de clúster después de establecer el valor nuevo_

### <a name="open-the-windows-firewall-probe-port"></a><a name="4498c707-86c0-4cde-9c69-058a7ab8c3ac"></a> Apertura del puerto de sondeo de Firewall de Windows

Abra el puerto de sondeo de firewall de Windows en ambos nodos del clúster. El siguiente script permite abrir un puerto de sondeo de firewall de Windows. Actualice las variables de PowerShell de su entorno.

  ```powershell
  $ProbePort = 62000   # ProbePort of the Azure internal load balancer

  New-NetFirewallRule -Name AzureProbePort -DisplayName "Rule for Azure Probe Port" -Direction Inbound -Action Allow -Protocol TCP -LocalPort $ProbePort
  ```

**ProbePort** está establecido en **62000**. Ahora puede tener acceso al recurso compartido de archivos \\\ascsha-clsap\sapmnt desde otros hosts, como ascsha-dbas.

## <a name="install-the-database-instance"></a><a name="85d78414-b21d-4097-92b6-34d8bcb724b7"></a> Instalación de la instancia de base de datos

Para instalar la instancia de base de datos, siga el proceso que se describe en la documentación de instalación de SAP.

## <a name="install-the-second-cluster-node"></a><a name="8a276e16-f507-4071-b829-cdc0a4d36748"></a> Instalación del segundo nodo del clúster

Para instalar el segundo clúster, siga los pasos que se describen en la guía de instalación de SAP.

## <a name="change-the-start-type-of-the-sap-ers-windows-service-instance"></a><a name="094bc895-31d4-4471-91cc-1513b64e406a"></a> Cambio del tipo de inicio del servicio de Windows para la instancia de ERS de SAP

Cambie el tipo de inicio del servicio o servicios de Windows de ERS de SAP a **Automático (inicio retrasado)** en ambos nodos del clúster.

![Ilustración 5: Cambio del tipo de servicio de la instancia de ERS de SAP a automático retrasado][sap-ha-guide-figure-3050]

_**Ilustración 5:** Cambio del tipo de servicio de la instancia de ERS de SAP a automático retrasado_

## <a name="install-the-sap-primary-application-server"></a><a name="2477e58f-c5a7-4a5d-9ae3-7b91022cafb5"></a> Instalación del servidor de aplicaciones principal de SAP

Instale la instancia del servidor de aplicaciones principal (PAS) \<SID\>-di-0 en la máquina virtual que designó para hospedar el PAS. No hay ninguna dependencia en Azure. No hay ninguna configuración específica de DataKeeper.

## <a name="install-the-sap-additional-application-server"></a><a name="0ba4a6c1-cc37-4bcf-a8dc-025de4263772"></a> Instalación del servidor de aplicaciones adicional de SAP

Instale un servidor de aplicaciones adicional (AAS) de SAP en todas las máquinas virtuales que designó para hospedar una instancia de servidores de aplicaciones de SAP. Por ejemplo, en \<SID\>-di-1 a \<SID\>-di-&lt;n&gt;.

> [!NOTE]
> Esta acción finaliza la instalación de un sistema SAP NetWeaver de alta disponibilidad. Después, continúe con las pruebas de conmutación por error.
>


## <a name="test-the-sap-ascsscs-instance-failover-and-sios-replication"></a><a name="18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9"></a> Prueba de la conmutación por error de la instancia de ASCS/SCS de SAP y de la replicación de SIOS
Es fácil probar y supervisar la conmutación por error de la instancia de ASCS/SCS de SAP y la replicación de disco de SIOS mediante el uso del Administrador de clústeres de conmutación por error, la interfaz de usuario de SIOS DataKeeper y la herramienta de configuración.

### <a name="sap-ascsscs-instance-is-running-on-cluster-node-a"></a><a name="65fdef0f-9f94-41f9-b314-ea45bbfea445"></a> La instancia de ASCS/SCS de SAP se ejecuta en el nodo de clúster A

El grupo de clústeres SAP PR1 se ejecuta en el nodo de clúster A. Por ejemplo, en pr1-ascs-0. Asigne la unidad de disco compartido S, que forma parte del grupo de clústeres SAP PR1, al nodo de clúster A. La instancia de ASCS/SCS también usa la unidad de disco S. 

![Figura 6: Administrador de clústeres de conmutación por error: El grupo de clústeres \<SID\> de SAP se ejecuta en el nodo del clúster A][sap-ha-guide-figure-5000]

_**Figura 6:** Administrador de clústeres de conmutación por error: El grupo de clústeres \<SID\> de SAP se ejecuta en el nodo del clúster A_.

En la interfaz de usuario de SIOS DataKeeper y la herramienta de configuración, puede ver que los datos del disco compartido se replican sincrónicamente desde la unidad de volumen de origen S en el nodo de clúster A. Por ejemplo, de pr1-ascs-0 [10.0.0.40] a pr1-ascs-1 [10.0.0.41].

![Figura 7: En SIOS DataKeeper, replique el volumen local desde el nodo de clústeres A al nodo de clústeres B][sap-ha-guide-figure-5001]

_**Figura 7:** En SIOS DataKeeper, replique el volumen local desde el nodo de clústeres A al nodo de clústeres B_

### <a name="failover-from-node-a-to-node-b"></a><a name="5e959fa9-8fcd-49e5-a12c-37f6ba07b916"></a> Conmutación por error del nodo A al nodo B

1. Elija una de estas opciones para iniciar una conmutación por error del grupo de clústeres \<SID\> de SAP desde el nodo del clúster A al B:
   - Administrador de clústeres de conmutación por error  
   - PowerShell del clúster de conmutación por error

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPClusterGroup = "SAP $SAPSID"
   Move-ClusterGroup -Name $SAPClusterGroup

   ```
2. Reinicie el nodo del clúster A dentro del sistema operativo invitado Windows. Al hacerlo, se inicia una conmutación automática por error del grupo de clústeres \<SID\> de SAP del nodo A al B.  
3. Reinicie el nodo del clúster A desde Azure Portal. Al hacerlo, se inicia una conmutación automática por error del grupo de clústeres \<SID\> de SAP del nodo A al B.  
4. Reinicie el nodo del clúster A mediante Azure PowerShell. Al hacerlo, se inicia una conmutación automática por error del grupo de clústeres \<SID\> de SAP del nodo A al B.

   Después de la conmutación por error, el grupo de clústeres \<SID\> de SAP se ejecuta en el nodo del clúster B. Por ejemplo, en pr1-ascs-1.

   ![Figura 8: En el Administrador de clústeres de conmutación por error, el grupo de clústeres \<SID\> de SAP se ejecuta en el nodo del clúster B][sap-ha-guide-figure-5002]

   _**Figura 8**: En el Administrador de clústeres de conmutación por error, el grupo de clústeres \<SID\> de SAP se ejecuta en el nodo del clúster B._

   Ahora, el disco compartido está montado en el nodo del clúster B. SIOS DataKeeper replica datos desde la unidad de volumen de origen S en el nodo del clúster B a la unidad de volumen de destino S en el nodo del clúster A. Por ejemplo, de pr1-ascs-1 [10.0.0.41] a pr1-ascs-0 [10.0.0.40].

   ![Figura 9: SIOS DataKeeper replica el volumen local desde el nodo del clúster B al nodo del clúster A][sap-ha-guide-figure-5003]

   _**Figura 9:** SIOS DataKeeper replica el volumen local desde el nodo del clúster B al nodo del clúster A_
