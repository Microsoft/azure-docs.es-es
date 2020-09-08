---
title: Arquitecturas de las soluciones con Azure NetApp Files | Microsoft Docs
description: Proporciona referencias a los procedimientos recomendados de las arquitecturas de soluciones que usa Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: b-juche
ms.openlocfilehash: e7bdf6ce35e2649a068de234b4b52e13efe72c44
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690551"
---
# <a name="solution-architectures-using-azure-netapp-files"></a>Arquitecturas de las soluciones con Azure NetApp Files
En este artículo se proporcionan referencias a los procedimientos recomendados que pueden ayudarle a comprender las arquitecturas de soluciones para usar Azure NetApp Files.  

El diagrama siguiente resume las categorías de arquitecturas de soluciones que Azure NetApp Files ofrece:

![Categorías de la arquitectura de la solución](../media/azure-netapp-files/solution-architecture-categories.png)

## <a name="linux-oss-apps-and-database-solutions"></a>Aplicaciones OSS de Linux y soluciones de base de datos

En esta sección se proporcionan referencias para las soluciones de las bases de datos y las aplicaciones OSS de Linux. 

### <a name="oracle"></a>Oracle

* [Guía de procedimientos recomendados para la implementación de Oracle en Azure con Azure NetApp Files](https://www.netapp.com/us/media/tr-4780.pdf)
* [Imágenes de VM de Oracle y su implementación en Microsoft Azure: Opciones de configuración de almacenamiento compartido](https://docs.microsoft.com/azure/virtual-machines/workloads/oracle/oracle-vm-solutions#shared-storage-configuration-options)
* [Ventajas del uso de Azure NetApp Files con Oracle Database](solutions-benefits-azure-netapp-files-oracle-database.md)

## <a name="windows-apps-and-sql-server-solutions"></a>Aplicaciones de Windows y soluciones de SQL Server

En esta sección se proporcionan referencias para las aplicaciones de Windows y soluciones de SQL Server.

### <a name="file-sharing-and-global-file-caching"></a>Uso compartido de archivos y almacenamiento en caché global de archivos

* [¿Quiere crear su propia instancia de Azure NFS? Cómo lidiar con los recursos compartidos de archivos Linux en la nube](https://cloud.netapp.com/blog/ma-anf-blg-build-your-own-linux-nfs-file-shares)
* [Caché de archivos globales/implementación de Azure NetApp Files](https://youtu.be/91LKb1qsLIM)

### <a name="sql-server"></a>SQL Server

* [Implementar SQL Server a través de SMB con Azure NetApp Files](https://www.youtube.com/watch?v=x7udfcYbibs)
* [Implementar el clúster de conmutación por error AlwaysOn de SQL Server en SMB con Azure NetApp Files](https://www.youtube.com/watch?v=zuNJ5E07e8Q)
* [Implementar grupos de disponibilidad AlwaysOn con Azure NetApp Files](https://www.youtube.com/watch?v=y3VQmzzeyvc)

## <a name="sap-on-azure-solutions"></a>Soluciones de SAP en Azure

En esta sección se proporcionan referencias de soluciones de SAP en Azure. 

### <a name="generic-sap-and-sap-netweaver"></a>SAP genérico y SAP NetWeaver 

* [Aplicaciones SAP en Microsoft Azure usando Azure NetApp Files](https://www.netapp.com/us/media/tr-4746.pdf)
* [Alta disponibilidad de SAP NetWeaver en VM de Azure en SUSE Linux Enterprise Server con Azure NetApp Files para las aplicaciones de SAP](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-suse-netapp-files)
* [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en Red Hat Enterprise Linux con Azure NetApp Files para aplicaciones SAP](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-rhel-netapp-files)
* [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en Windows con Azure NetApp Files (SMB) para aplicaciones SAP](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-windows-netapp-files-smb)
* [Alta disponibilidad para SAP NetWeaver en VM de Azure en Red Hat Enterprise Linux para SAP Applications: guía de varios SID](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-rhel-multi-sid)

### <a name="sap-hana"></a>SAP HANA 

* [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-vm-operations-storage)
* [Alta disponibilidad del escalado vertical de SAP HANA con Azure NetApp Files en Red Hat Enterprise Linux](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-high-availability-netapp-files-red-hat)
* [Escalabilidad horizontal de SAP HANA con nodo en espera en máquinas virtuales de Azure con Azure NetApp Files en SUSE Linux Enterprise Server](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-scale-out-standby-netapp-files-suse)
* [Escalabilidad horizontal de SAP HANA con nodo en espera en máquinas virtuales de Azure con Azure NetApp Files en Red Hat Enterprise Linux](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-scale-out-standby-netapp-files-rhel)

### <a name="sap-tech-community-and-blog-posts"></a>Publicaciones de blog y comunidad tecnológica de SAP 

* [Azure NetApp Files: copia de seguridad SAP HANA en segundos](https://blog.netapp.com/azure-netapp-files-sap-hana-backup-in-seconds/)
* [Azure NetApp Files: restaure la base de datos HANA a partir de una copia de seguridad de instantánea](https://blog.netapp.com/azure-netapp-files-backup-sap-hana)
* [Azure NetApp Files: Descarga de copias de seguridad de SAP HANA con sincronización en la nube](https://blog.netapp.com/azure-netapp-files-sap-hana)
* [Acelere las copias del sistema SAP HANA con Azure NetApp Files](https://blog.netapp.com/sap-hana-faster-using-azure-netapp-files/)
* [Volúmenes en la nube ONTAP y Azure NetApp Files: simplificación de la migración del sistema SAP HANA](https://blog.netapp.com/cloud-volumes-ontap-and-azure-netapp-files-sap-hana-system-migration-made-easy/)

## <a name="virtual-desktop-infrastructure-solutions"></a>Soluciones de infraestructura de escritorio virtual

En esta sección se proporcionan referencias para las soluciones de infraestructura de escritorio virtual.

### <a name="windows-virtual-desktop"></a>Windows Virtual Desktop

* [Ventajas de usar Azure NetApp Files con Windows Virtual Desktop](solutions-windows-virtual-desktop.md)
* [Opciones de almacenamiento para los contenedores de perfiles de FSLogix de Windows Virtual Desktop](https://docs.microsoft.com/azure/virtual-desktop/store-fslogix-profile#azure-platform-details)
* [Creación de un contenedor de perfiles de FSLogix para un grupo host mediante Azure NetApp Files](https://docs.microsoft.com/azure/virtual-desktop/create-fslogix-profile-container)
* [Windows Virtual Desktop a escala empresarial](https://docs.microsoft.com/azure/architecture/example-scenario/wvd/windows-virtual-desktop)

## <a name="hpc-solutions"></a>Soluciones HPC

En esta sección se proporcionan referencias para las soluciones de informática de alto rendimiento (HPC). 

### <a name="generic-hpc"></a>HPC genérico

* [Azure NetApp Files: Cómo sacar el máximo partido del almacenamiento en la nube](https://cloud.netapp.com/hubfs/Resources/ANF%20PERFORMANCE%20TESTING%20IN%20TEMPLATE.pdf)
* [Ejecución de cargas de trabajo de MPI con Azure Batch y Azure NetApp Files](https://azure.microsoft.com/resources/run-mpi-workloads-with-azure-batch-and-azure-netapp-files/)
* [Azure Cycle Cloud: Entornos de HPC de CycleCloud en Azure NetApp Files](https://docs.microsoft.com/azure/cyclecloud/overview)

### <a name="oil-and-gas"></a>Petróleo y gas

* [Informática de alto rendimiento (HPC): Petróleo y gas en Azure](https://techcommunity.microsoft.com/t5/azure-global/high-performance-computing-hpc-oil-and-gas-in-azure/ba-p/824926)
* [Ejecución de software de simulación de depósitos en Azure](https://docs.microsoft.com/azure/architecture/example-scenario/infrastructure/reservoir-simulation)

### <a name="electronic-design-automation-eda"></a>Automatización de diseño electrónico (EDA)

* [Ventajas de usar Azure NetApp Files para la automatización de diseños electrónicos](solutions-benefits-azure-netapp-files-electronic-design-automation.md)
* [Azure CycleCloud: Laboratorio de HPC de EDA con Azure NetApp Files](https://github.com/Azure/cyclecloud-hands-on-labs/blob/master/EDA/README.md)

### <a name="analytics"></a>Análisis

* [Azure NetApp Files: Un nuevo sistema de archivos compartidos que se usará con la cuadrícula SAS en Microsoft Azure](https://communities.sas.com/t5/Architecture/Azure-NetApp-Files-A-new-shared-file-system-to-use-with-SAS-Grid/m-p/606978)

## <a name="azure-platform-services-solutions"></a>Soluciones de servicios de la plataforma Azure

En esta sección se proporcionan soluciones para los servicios de la plataforma Azure. 

### <a name="azure-kubernetes-services-and-kubernetes"></a>Azure Kubernetes Service y Kubernetes

* [Integración de Azure NetApp Files con Azure Kubernetes Service](https://docs.microsoft.com/azure/aks/azure-netapp-files)
* [Rendimiento Kubernetes fuera de este mundo en Azure con Azure NetApp Files](https://cloud.netapp.com/blog/ma-anf-blg-configure-kubernetes-openshift)
* [Trident: orquestador de almacenamiento para contenedores](https://netapp-trident.readthedocs.io/en/stable-v20.04/kubernetes/operations/tasks/backends/anf.html)

### <a name="azure-batch"></a>Azure Batch

* [Ejecución de cargas de trabajo de MPI con Azure Batch y Azure NetApp Files](https://azure.microsoft.com/resources/run-mpi-workloads-with-azure-batch-and-azure-netapp-files/)
 
