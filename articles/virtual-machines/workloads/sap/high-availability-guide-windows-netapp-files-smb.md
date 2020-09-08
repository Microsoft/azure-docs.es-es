---
title: Alta disponibilidad de VM de Azure para SAP NW en Windows con Azure NetApp Files (SMB) | Microsoft Docs
description: Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en Windows con Azure NetApp Files (SMB) para aplicaciones SAP
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
ms.date: 08/12/2020
ms.author: radeltch
ms.openlocfilehash: cd974377637f535383c4e099ac408bea88f887a4
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88853108"
---
# <a name="high-availability-for-sap-netweaver-on-azure-vms-on-windows-with-azure-netapp-filessmb-for-sap-applications"></a>Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en Windows con Azure NetApp Files (SMB) para aplicaciones SAP

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-azure-doc]:https://docs.microsoft.com/azure/azure-netapp-files/
[anf-avail-matrix]:https://azure.microsoft.com/global-infrastructure/services/?products=storage&regions=all
[anf-register]:https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register
[anf-sap-applications-azure]:https://www.netapp.com/us/media/tr-4746.pdf

[2205917]:https://launchpad.support.sap.com/#/notes/2205917
[1944799]:https://launchpad.support.sap.com/#/notes/1944799
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[1410736]:https://launchpad.support.sap.com/#/notes/1410736

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[suse-ha-guide]:https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
[suse-drbd-guide]:https://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha_techguides/book_sleha_techguides.html
[suse-ha-12sp3-relnotes]:https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP3/

[template-multisid-xscs]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[template-converged]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-converged-md%2Fazuredeploy.json
[template-file-server]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-file-server-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability.md
[nfs-ha]:high-availability-guide-suse-nfs.md

En este artículo se describe cómo implementar, configurar las máquinas virtuales, instalar el marco de trabajo de clúster e instalar un sistema SAP NetWeaver 7.50 de alta disponibilidad en máquinas virtuales Windows mediante [SMB](/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) en [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md).  

El nivel de la base de datos no se trata en detalle en este artículo. Se supone que ya se ha creado la [red virtual](../../../virtual-network/virtual-networks-overview.md) de Azure.  

Lea primero las notas y los documentos de SAP siguientes:

* [Documentación sobre Azure NetApp Files][anf-azure-doc] 
* La nota de SAP [1928533][1928533], que contiene:  
  * Una lista de los tamaños de máquina virtual de Azure que se admiten para la implementación de software de SAP
  * Información importante sobre la capacidad para los tamaños de máquina virtual de Azure
  * Software de SAP admitido y combinaciones de sistema operativo y base de datos
  * Versión de kernel de SAP necesaria para Windows en Microsoft Azure
* La nota de SAP [2015553][2015553] enumera los requisitos previos para las implementaciones de software de SAP admitidas por SAP en Azure.
* La nota de SAP [2178632][2178632] contiene información detallada sobre todas las métricas de supervisión notificadas para SAP en Azure.
* La nota de SAP [1999351][1999351] contiene más soluciones de problemas de la extensión de supervisión mejorada de Azure para SAP.
* La nota de SAP [2287140](https://launchpad.support.sap.com/#/notes/2287140) enumera los requisitos previos para la característica de CA compatible con SAP del protocolo SMB 3.x.
* La nota de SAP [2802770](https://launchpad.support.sap.com/#/notes/2802770) tiene información de solución de problemas para el AL11 de transacciones de SAP de ejecución lenta en Windows 2012 y 2016.
* La nota de SAP [1911507](https://launchpad.support.sap.com/#/notes/1911507) tiene información sobre la característica de conmutación por error transparente para un recurso compartido de archivos en Windows Server con el protocolo SMB 3.0.
* La nota de SAP [662452](https://launchpad.support.sap.com/#/notes/662452) tiene una recomendación (desactivación de la generación de nombres 8.3) para solucionar errores o el rendimiento del sistema de archivos deficiente durante el acceso a los datos.
* [Instalación de alta disponibilidad de SAP NetWeaver en un clúster de conmutación por error de Windows y un recurso compartido de archivos para instancias de ASCS/SCS de SAP en Azure](./sap-high-availability-installation-wsfc-file-share.md) 
* [Escenarios y arquitectura de alta disponibilidad de Azure Virtual Machines para SAP NetWeaver](./sap-high-availability-architecture-scenarios.md)
* [Agregar puerto de sondeo en la configuración de clúster de ASCS](sap-high-availability-installation-wsfc-file-share.md)
* [Instalación de una instancia de (A) SCS en un clúster de conmutación por error](https://www.sap.com/documents/2017/07/f453332f-c97c-0010-82c7-eda71af511fa.html)
* [Cree un volumen SMB para Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes-smb.md#requirements-for-active-directory-connections)
* [Aplicaciones de NetApp SAP en Microsoft Azure que usan Azure NetApp Files][anf-sap-applications-azure]

## <a name="overview"></a>Información general

SAP ha desarrollado un nuevo enfoque y alternativas para agrupar en clústeres los discos compartidos, a fin de agrupar una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows. En lugar de usar discos compartidos de clúster, puede usar un recurso compartido de archivos SMB para implementar archivos de host global de SAP. Azure NetApp Files admite SMBv3 (junto con NFS) con la ACL de NTFS mediante Active Directory. Azure NetApp Files es automáticamente de alta disponibilidad (ya que es un servicio PaaS). Estas características hacen de Azure NetApp Files una opción excelente para hospedar el recurso compartido de archivos SMB para SAP global.  
Se admite tanto [Azure Active Directory (AD) Domain Services](../../../active-directory-domain-services/overview.md) como [Active Directory Domain Services (AD DS)](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview). Puede usar controladores de dominio de Active Directory existentes con Azure NetApp Files. Los controladores de dominio pueden estar en Azure como máquinas virtuales o de forma local mediante ExpressRoute o VPN de sitio a sitio. En este artículo, se usará el controlador de dominio en una máquina virtual de Azure.  
La alta disponibilidad de los servicios centrales de SAP Netweaver requiere almacenamiento compartido. Para lograrlo en Windows, hasta ahora era necesario crear un clúster de SOFS o usar el disco compartido de clúster s/w como SIOS. Ahora ya es posible lograr alta disponibilidad en SAP NetWeaver mediante el uso de almacenamiento compartido implementado en Azure NetApp Files. El uso de Azure NetApp Files para el almacenamiento compartido elimina la necesidad de SOFS o SIOS.  

> [!NOTE]
> La agrupación en clústeres de instancias de ASCS/SCS de SAP con un recurso compartido de archivos es compatible con SAP NetWeaver 7.40 (y versiones posteriores), con el kernel de SAP 7.49 (y versiones posteriores).  

![Arquitectura de alta disponibilidad de ASCS/SCS de SAP con recurso compartido de SMB](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb.png)

Los requisitos previos para un recurso compartido de archivos SMB son:
* Protocolo SMB 3.0 (o posterior).
* Capacidad para establecer Active Directory listas de control de acceso (ACL) para Active Directory grupos de usuarios y el objeto de equipo computer$.
* El recurso compartido de archivos debe estar habilitado para alta disponibilidad.

El recurso compartido de los servicios centrales de SAP en esta arquitectura de referencia se ofrece mediante Azure NetApp Files:

![Detalles de la arquitectura de alta disponibilidad de ASCS/SCS de SAP con recurso compartido de SMB](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb-detail.png)

## <a name="create-and-mount-smb-volume-for-azure-netapp-files"></a>Crear y montar el volumen SMB para Azure NetApp Files

Siga los pasos que se indican a continuación para preparar el uso de Azure NetApp Files.  

1. Siga los pasos para [Registrarse en Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md)  
2. Cree una cuenta de Azure NetApp, al seguir los pasos descritos en [Creación de una cuenta de NetApp](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md)  
3. Configure el grupo de capacidad siguiendo las instrucciones en [Configurar un grupo de capacidad](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md)
4. Los recursos Azure NetApp Files deben residir en una subred delegada. Siga las instrucciones en [Delegar una subred en Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md) para crear una subred delegada.  

   > [!IMPORTANT]
   > Debe crear las conexiones de Active Directory antes de crear un volumen SMB. Revise los [requisitos para las conexiones de Active Directory](../../../azure-netapp-files/azure-netapp-files-create-volumes-smb.md#requirements-for-active-directory-connections).  

5. Cree una conexión Active Directory, como se describe en [Crear una conexión de Active Directory](../../../azure-netapp-files/azure-netapp-files-create-volumes-smb.md#create-an-active-directory-connection)  
6. Cree el volumen SMB Azure NetApp Files SMB al seguir las instrucciones en [Agregar un volumen SMB](../../../azure-netapp-files/azure-netapp-files-create-volumes-smb.md#add-an-smb-volume)  
7. Monte el volumen SMB en la máquina virtual de Windows.

> [!TIP]
> Puede encontrar instrucciones sobre cómo montar el volumen de Azure NetApp Files, si navega en [Azure Portal](https://portal.azure.com/#home) al objeto Azure NetApp Files, haga clic en la hoja **Volúmenes** y después en **instrucciones de montaje**.  

## <a name="prepare-the-infrastructure-for-sap-ha-by-using-a-windows-failover-cluster"></a>Preparación de la infraestructura para la alta disponibilidad de SAP mediante un clúster de conmutación por error de Windows 

1. [Definición de reglas predeterminadas de equilibrio de carga de ASCS/SCS para el equilibrador de carga interno de Azure](./sap-high-availability-infrastructure-wsfc-shared-disk.md#fe0bd8b5-2b43-45e3-8295-80bee5415716)
2. [Incorpore máquinas virtuales de Windows al dominio](./sap-high-availability-infrastructure-wsfc-shared-disk.md#e69e9a34-4601-47a3-a41c-d2e11c626c0c).
3. [Incorporación de entradas del registro en ambos nodos del clúster de la instancia de ASCS/SCS de SAP](./sap-high-availability-infrastructure-wsfc-shared-disk.md#661035b2-4d0f-4d31-86f8-dc0a50d78158)
4. [Configuración de un clúster de conmutación por error de Windows Server para una instancia de ASCS/SCS de SAP](./sap-high-availability-infrastructure-wsfc-shared-disk.md#0d67f090-7928-43e0-8772-5ccbf8f59aab)
5. Si usa Windows Server 2016, se recomienda que configure [testigo en la nube de Azure](/windows-server/failover-clustering/deploy-cloud-witness).


## <a name="install-sap-ascs-instance-on-both-nodes"></a>Instalación de la instancia de ASCS de SAP en ambos nodos

Necesita el siguiente software de SAP:
   * Versión de la herramienta de instalación SAP Software Provisioning Manager (SWPM) SPS25 (o posterior).
   * Kernel de SAP 7.49 o posterior
   * Cree un nombre de host virtual (nombre de red en clúster) para la instancia de ASCS/SCS de SAP en clúster, tal como se describe en [Crear un nombre de host virtual para la instancia de ASCS/SCS de SAP en clúster](./sap-high-availability-installation-wsfc-shared-disk.md#a97ad604-9094-44fe-a364-f89cb39bf097).

> [!NOTE]
> La agrupación en clústeres de instancias de ASCS/SCS de SAP con un recurso compartido de archivos es compatible con SAP NetWeaver 7.40 (y versiones posteriores), con el kernel de SAP 7.49 (y versiones posteriores).  

### <a name="install-an-ascsscs-instance-on-the-first-ascsscs-cluster-node"></a>Instalación de instancias de ASCS/SCS en el primer nodo de clúster de ASCS/SCS

1. Instale una instancia de ASCS/SCS de SAP en el primer nodo de clúster. Inicie la herramienta de instalación de SAP SWPM y, luego, vaya a: **Producto** > **DBMS** > Instalación > Servidor de aplicaciones ABAP (o Java) > Sistema de alta disponibilidad > instancia de ASCS/SCS > Primer nodo de clúster.  

2. Seleccione **Clúster de recursos compartidos de archivos** como la configuración del recurso compartido de clúster en SWPM.  
3. Cuando se le solicite en el paso **Parámetros de clúster del sistema SAP**, escriba el nombre de host para el recurso compartido de Azure NetApp Files SMB que ya ha creado como **Nombre de host de recurso compartido de archivos**.  En este ejemplo, el nombre de host del recurso compartido de SMB es **anfsmb-9562**. 

   > [!IMPORTANT]
   > Si el comprobador de requisitos previos da como resultado SWPM muestra la condición de la característica disponibilidad continua no cumplida, se puede solucionar al seguir las instrucciones de [Mensaje de error retrasado al intentar obtener acceso a una carpeta compartida que ya no existe en Windows](https://support.microsoft.com/help/2820470/delayed-error-message-when-you-try-to-access-a-shared-folder-that-no-l).  

   > [!TIP]
   > Si el comprobador de requisitos previos da como resultado SWPM muestra que no se cumple la condición de tamaño de intercambio, puede ajustar el tamaño de intercambio si navega a Mi PC> Propiedades del sistema> Configuración de rendimiento> Avanzado> Memoria virtual> Cambio.  

4. Configure un recurso de clúster de SAP, el `SAP-SID-IP` puerto de sondeo, mediante PowerShell. Ejecute esta configuración en uno de los nodos de clúster de ASCS/SCS de SAP, como se describe en [Configurar el puerto de sondeo](./sap-high-availability-installation-wsfc-shared-disk.md#10822f4f-32e7-4871-b63a-9b86c76ce761).

### <a name="install-an-ascsscs-instance-on-the-second-ascsscs-cluster-node"></a>Instalación de instancias de ASCS/SCS en el segundo nodo de clúster de ASCS/SCS

1. Instale una instancia de ASCS/SCS de SAP en el segundo nodo de clúster. Inicie la herramienta de instalación de SAP SWPM y, luego, vaya a **Producto** > **DBMS** > Instalación > Servidor de aplicaciones ABAP (o Java) > Sistema de alta disponibilidad > instancia de ASCS/SCS > Nodo de clúster adicional.  

### <a name="install-a-dbms-instance-and-sap-application-servers"></a>Instalación de instancias de DBMS y servidores de aplicaciones SAP

Complete la instalación de SAP mediante la instalación de:

   * Una instancia de DBMS  
   * Un servidor principal de aplicaciones SAP  
   * Un servidor adicional de aplicaciones SAP  

## <a name="test-the-sap-ascsscs-instance-failover"></a>Prueba de la conmutación por error de la instancia de ASCS/SCS de SAP 

### <a name="fail-over-from-cluster-node-a-to-cluster-node-b-and-back"></a>Conmutar por error desde el nodo de clúster a al nodo de clúster B y atrás
En este escenario de prueba, haremos referencia al nodo de clúster sapascs1 como nodo A y al nodo de clúster sapascs2 como nodo B.

1. Compruebe que los recursos del clúster se ejecutan en el nodo A. ![Figura 1: Recursos de clúster de conmutación por error de Windows Server que se ejecutan en el nodo A antes de la prueba de conmutación](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb-figure-1.png)  

2. Reinicie el nodo de clúster A. Los recursos de clúster de SAP se moverán al nodo del clúster B. ![Figura 2: Recursos de clúster de conmutación por error de Windows Server en ejecución en el nodo B después de la prueba de conmutación por error](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb-figure-2.png)  


## <a name="lock-entry-test"></a>Prueba de entrada de bloqueo

1. Compruebe que el servidor de replicación de puesta en cola de SAP (ERS) está activo  
2. Inicie sesión en el sistema SAP, ejecute la transacción SU01 y abra un identificador de usuario en modo de cambio. Esto generará una entrada de bloqueo de SAP.  
3. Cuando haya iniciado sesión en el sistema SAP, puede mostrar la entrada de bloqueo desplazándose a la transacción ST12.  
4. Conmute por error los recursos de ASCS del nodo de clúster A al nodo de clúster B.  
5. Compruebe que se conserva la entrada de bloqueo, generada antes de la conmutación por error de los recursos de clúster de ASCS/SCS de SAP.  

![Ilustración 3: La entrada de bloqueo se conserva después de la prueba de conmutación por error](./media/virtual-machines-shared-sap-high-availability-guide/high-availability-windows-azure-netapp-files-smb-figure-3.png)  

Para más información, consulte [Solución de problemas de la conmutación por error de puesta en cola en ASCS con ERS](https://wiki.scn.sap.com/wiki/display/SI/Troubleshooting+for+Enqueue+Failover+in+ASCS+with+ERS)
## <a name="next-steps"></a>Pasos siguientes

* [Planeamiento e implementación de Azure Virtual Machines para SAP][planning-guide]
* [Implementación de Azure Virtual Machines para SAP][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP][dbms-guide]
* Para saber cómo establecer una estrategia de alta disponibilidad y recuperación ante desastres de SAP 
* HANA en Azure (instancias grandes), consulte [Alta disponibilidad y recuperación ante desastres de SAP HANA en Azure (instancias grandes)](hana-overview-high-availability-disaster-recovery.md).
* Para más información sobre cómo establecer la alta disponibilidad y planear la recuperación ante desastres de SAP HANA en Azure Virtual Machines, consulte [Alta disponibilidad de SAP HANA en las máquinas virtuales de Azure][sap-hana-ha].
