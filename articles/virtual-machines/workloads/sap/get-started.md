---
title: Empezar a trabajar con SAP en máquinas virtuales de Azure | Microsoft Docs
description: Obtenga información acerca de las soluciones SAP que se ejecutan en máquinas virtuales (VM) en Microsoft Azure
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: ad8e5c75-0cf6-4564-ae62-ea1246b4e5f2
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 05/15/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: f14110cec0650927f74dfe3a38f06187c6df39de
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65792658"
---
# <a name="use-azure-to-host-and-run-sap-workload-scenarios"></a>Use Azure para hospedar y ejecutar escenarios de carga de trabajo SAP

Cuando se usa Microsoft Azure, puede ejecutar forma confiable los escenarios y las cargas de trabajo SAP críticas en una plataforma escalable, compatible y demostrado en la empresa. Obtenga la escalabilidad, flexibilidad y ahorro de costos de Azure. Con la ampliación de la asociación entre Microsoft y SAP, puede ejecutar aplicaciones de SAP en escenarios de desarrollo y pruebas y producción en Azure y serán totalmente compatibles. De SAP NetWeaver a SAP S/4HANA, BI de SAP en Linux a Windows y SAP HANA a SQL, tenemos todo cubierto.

Además de hospedar escenarios de SAP NetWeaver con diferentes DBMS en Azure, puede hospedar otros escenarios de carga de trabajo SAP, como BI de SAP en Azure. 

La unicidad de Azure para SAP HANA es una oferta que distingue a Azure. Para habilitar el hospedaje de más memoria y CPU más exigentes recursos SAP escenarios que implican SAP HANA, Azure ofrece el uso de hardware de reconstrucción completa dedicado de cliente. Use esta solución para ejecutar las implementaciones de SAP HANA que requieren hasta 24 TB (120 TB de escalado horizontal) de memoria para otras cargas de trabajo de SAP HANA o S/4HANA. 

Hospedaje de escenarios de carga de trabajo SAP en Azure también puede crear requisitos de integración de identidades y de inicio de sesión único. Esta situación puede producirse al usar Azure Active Directory (Azure AD) para conectar los diferentes componentes SAP y SAP software-como-servicio (SaaS) o las ofertas de plataforma como servicio (PaaS). Una lista de dicha integración y escenarios de inicio de sesión único con Azure AD y entidades SAP se describe y documenta en la sección "integración de identidades AAD SAP y single sign-on".

## <a name="latest-changes"></a>Últimos cambios

- Versión de [controlar instancias grandes de HANA de Azure a través del portal de Azure](hana-li-portal.md)

- Versión de [alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server con Azure Files de NetApp para las aplicaciones de SAP](high-availability-guide-suse-netapp-files.md)

- Aclaración en **net.ipv4.tcp_timestamps de parámetro del sistema operativo Linux** equilibrador de carga de configuración junto con una instancia de Azure

- Versión de [las configuraciones de carga de trabajo SAP con zonas de disponibilidad de Azure](sap-ha-availability-zones.md)

- Publicación de la [lista de comprobación de planeación e implementación de carga de trabajo de SAP](sap-deployment-checklist.md)




## <a name="sap-hana-on-azure-large-instances"></a>SAP HANA en Azure (instancias grandes)

Una serie de documentos le guía a través de SAP HANA en Azure (instancias grandes), o bien, para abreviar, instancias grandes de HANA. Para obtener información acerca de las siguientes áreas de instancias grandes de HANA, consulte:

- [Introducción de SAP HANA en Azure (instancias grandes)](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture)
- [Arquitectura de SAP HANA en Azure (instancias grandes)](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-architecture)
- [Infraestructura y conectividad con SAP HANA en Azure (instancias grandes)](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-infrastructure-connectivity)
- [Instalación de SAP HANA en Azure (instancias grandes)](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-installation)
- [Alta disponibilidad y recuperación ante desastres de SAP HANA en Azure (instancias grandes)](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery)
- [Solución de problemas y supervisión de SAP HANA en Azure (instancias grandes)](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/troubleshooting-monitoring)

Pasos siguientes:

- Lectura [introducción y arquitectura de SAP HANA en Azure (instancias grandes)](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture)



## <a name="sap-hana-on-azure-virtual-machines"></a>SAP HANA en máquinas virtuales de Azure
Esta sección de la documentación trata diferentes aspectos de SAP HANA. Como requisito previo, debe estar familiarizado con los principales servicios de Azure que proporcionan servicios básicos de IaaS de Azure. Por lo tanto, necesita conocimientos de redes, almacenamiento y proceso de Azure. Muchos de estos temas se tratan en SAP NetWeaver relacionan [Guía de planeación Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/planning-guide). 

Para obtener información sobre HANA en Azure, consulte los siguientes artículos y sus subarticles:

- [Inicio rápido: instalación manual de una única instancia de SAP HANA en máquinas virtuales de Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-get-started)
- [Implementación de SAP S/4HANA o BW/4HANA en Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/cal-s4h)
- [Configuraciones y operaciones de infraestructura de SAP HANA en Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-vm-operations)
- [Alta disponibilidad de SAP HANA para máquinas virtuales de Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-availability-overview)
- [Disponibilidad de SAP HANA dentro de una región de Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-availability-one-region)
- [Disponibilidad de SAP HANA entre regiones de Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-availability-across-regions)
- [Alta disponibilidad de SAP HANA en máquinas virtuales de Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-high-availability)
- [Guía de copia de seguridad de SAP HANA en Azure virtual machines](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-backup-guide)
- [Azure Backup de SAP HANA en el nivel de archivo](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-backup-file-level)
- [Copia de seguridad de SAP HANA basada en instantáneas de almacenamiento](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-backup-storage-snapshots)

 

## <a name="sap-netweaver-deployed-on-azure-virtual-machines"></a>SAP NetWeaver implementadas en Azure virtual machines
Esta sección enumeran la documentación de planeación e implementación de SAP NetWeaver y Business One en Azure. La documentación se centra en los conceptos básicos y el uso de las bases de datos no perteneciente a HANA con una carga de trabajo SAP en Azure. Los documentos y los artículos para lograr alta disponibilidad también son la base para lograr alta disponibilidad HANA en Azure, como:

- [SAP Business One en Azure virtual machines](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/business-one-azure)
- [Implementación de SAP IDES EHP7 SP3 para SAP ERP 6.0 en Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/cal-ides-erp6-erp7-sp3-sql)
- [Ejecute SAP NetWeaver en máquinas virtuales Linux SUSE de Microsoft Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/suse-quickstart)
- [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/planning-guide)
- [Implementación de Azure Virtual Machines para SAP NetWeaver](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/deployment-guide)
- [Proteger una implementación de aplicaciones multinivel de SAP NetWeaver mediante Site Recovery](https://docs.microsoft.com/azure/site-recovery/site-recovery-sap)
- [Conector de SAP LaMa para Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/lama-installation)

Para obtener información sobre las bases de datos de HANA no bajo una carga de trabajo SAP en Azure, consulte:

- [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_general)
- [Implementación de DBMS de Azure Virtual Machines de SQL Server para la carga de trabajo de SAP NetWeaver](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_sqlserver)
- [Implementación de DBMS de Azure Virtual Machines de Oracle para una carga de trabajo de SAP](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_oracle)
- [Implementación de DBMS de Azure Virtual Machines de IBM DB2 para una carga de trabajo de SAP](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_ibm)
- [Implementación de DBMS de Azure Virtual Machines de SAP ASE para una carga de trabajo de SAP](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_sapase)
- [Implementación de SAP MaxDB, caché en vivo y Content Server en máquinas virtuales de Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_maxdb)

Para obtener información sobre las bases de datos de SAP HANA en Azure, consulte la sección "SAP HANA en Azure virtual machines".

Para obtener información sobre la alta disponibilidad de una carga de trabajo SAP en Azure, consulte:

- [Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-guide-start)

Este documento se apunta a varios otros documentos de arquitectura y escenario. En los documentos de escenario más adelante, se proporcionan vínculos a documentos técnicos detallados que explican la implementación y configuración de los diferentes métodos de alta disponibilidad. Los distintos documentos que se muestran cómo establecer y configurar la alta disponibilidad para una carga de trabajo de SAP NetWeaver cubren los sistemas operativos Linux y Windows.


Para obtener información sobre la integración entre Azure Active Directory (Azure AD) y servicios de SAP y el inicio de sesión único, vea:

- [Tutorial: Integración de Azure Active Directory con SAP Cloud for Customer](https://docs.microsoft.com/azure/active-directory/saas-apps/sap-customer-cloud-tutorial?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutorial: Integración de Azure Active Directory con SAP Cloud Platform Identity Authentication](https://docs.microsoft.com/azure/active-directory/saas-apps/sap-hana-cloud-platform-identity-authentication-tutorial?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutorial: Integración de Azure Active Directory con SAP Cloud Platform](https://docs.microsoft.com/azure/active-directory/saas-apps/sap-hana-cloud-platform-tutorial?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutorial: Integración de Azure Active Directory con SAP NetWeaver](https://docs.microsoft.com/azure/active-directory/saas-apps/sap-netweaver-tutorial?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutorial: Integración de Azure Active Directory con SAP Business ByDesign](https://docs.microsoft.com/azure/active-directory/saas-apps/sapbusinessbydesign-tutorial?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutorial: Integración de Azure Active Directory con SAP HANA](https://docs.microsoft.com/azure/active-directory/saas-apps/saphana-tutorial?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [El entorno de S/4HANA: Fiori Launchpad SAML single sign-on con Azure AD](https://blogs.sap.com/2017/02/20/your-s4hana-environment-part-7-fiori-launchpad-saml-single-sing-on-with-azure-ad/)

Para obtener información sobre la integración de servicios de Azure en los componentes de SAP, consulte:

- [Uso de SAP HANA en Power BI Desktop](https://docs.microsoft.com/power-bi/desktop-sap-hana)
- [DirectQuery y SAP HANA](https://docs.microsoft.com/power-bi/desktop-directquery-sap-hana)
- [Uso del conector SAP BW en Power BI Desktop](https://docs.microsoft.com/power-bi/desktop-sap-bw-connector) 
- [Azure Data Factory ofrece integración de datos de SAP HANA y Business Warehouse](https://azure.microsoft.com/blog/azure-data-factory-offer-sap-hana-and-business-warehouse-data-integration)




