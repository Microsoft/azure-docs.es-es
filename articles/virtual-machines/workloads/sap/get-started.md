---
title: Introducción a SAP en máquinas virtuales de Azure | Microsoft Docs
description: Información sobre las soluciones SAP que se ejecutan en máquinas virtuales (VM) en Microsoft Azure
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: ad8e5c75-0cf6-4564-ae62-ea1246b4e5f2
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/04/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a882e3a21d5e1e99f6f9154fd2162071752b4499
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87800355"
---
# <a name="use-azure-to-host-and-run-sap-workload-scenarios"></a>Uso de Azure para hospedar y ejecutar escenarios de carga de trabajo de SAP

Cuando utiliza Microsoft Azure, puede ejecutar de forma confiable los escenarios y las cargas de trabajo de SAP críticas en una plataforma compatible, ampliable y de uso demostrado en la empresa. Consigue la escalabilidad, la flexibilidad y el ahorro de costos de Azure. Con la ampliación de la asociación entre Microsoft y SAP, puede ejecutar las aplicaciones SAP en escenarios de desarrollo, pruebas y producción en Azure y serán totalmente compatibles. De SAP NetWeaver a SAP S/4HANA, de BI de SAP en Linux a Windows y de SAP HANA a SQL, lo cubrimos todo.

Además de hospedar escenarios de SAP NetWeaver con los diferentes DBMS en Azure, puede hospedar otros escenarios de cargas de trabajo de SAP, como BI de SAP en Azure. 

La unicidad de Azure para SAP HANA es una oferta que distingue a Azure. Para habilitar el hospedaje de escenarios de SAP con más demanda de recursos de memoria y CPU que implican a SAP HANA, Azure ofrece el uso de hardware de reconstrucción completa dedicado para el cliente. Use esta solución para ejecutar las implementaciones de SAP HANA que requieren hasta 24 TB (120 TB de escalabilidad horizontal) de memoria para S/4HANA u otra carga de trabajo de SAP HANA. 

El hospedaje de escenarios de carga de trabajo de SAP en Azure también puede crear requisitos de integración de identidades y de inicio de sesión único. Esta situación puede producirse al usar Azure Active Directory (Azure AD) para conectar diferentes componentes y ofertas de software como servicio (SaaS) o plataforma como servicio (PaaS) de SAP. En la sección "Integración de identidades de AAD SAP e inicio de sesión único" se describe y documenta una lista de estos escenarios de integración e inicio de sesión único con entidades de Azure AD y SAP.

## <a name="changes-to-the-sap-workload-section"></a>Cambios en la sección de cargas de trabajo de SAP
Al final de este artículo se enumeran los cambios en los documentos de la sección sobre cargas de trabajo de SAP en Azure. Las entradas del registro de cambios se mantienen durante unos 180 días.

## <a name="you-want-to-know"></a>Más información
Si tiene preguntas específicas, vamos a apuntar a documentos o flujos específicos en esta sección de la página de inicio. Más información sobre los siguientes aspectos:

- Las VM de Azure y las unidades de instancias grandes de HANA se admiten en las versiones de software de SAP y en las versiones de sistema operativo. Lea el documento [Qué software de SAP es compatible con la implementación de Azure](./sap-supported-product-on-azure.md) para obtener respuestas y el proceso para buscar la información.
- Qué escenarios de implementación de SAP se admiten en las VM de Azure y las instancias grandes de HANA. Puede encontrar información sobre los escenarios admitidos en los siguientes documentos:
    - [Carga de trabajo de SAP en escenarios admitidos en máquinas virtuales de Azure](./sap-planning-supported-configurations.md)
    - [Escenarios admitidos para instancias grandes de HANA](./hana-supported-scenario.md)
- Para conocer qué servicios de Azure, tipos de VM de Azure y servicios de Azure Storage hay disponibles en las diferentes regiones de Azure, vea el sitio [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/) 
- ¿Se admiten marcos de alta disponibilidad de terceros, además de Windows y Pacemaker? Consulte la parte inferior de la [nota de soporte técnico de SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533).

 
## <a name="sap-hana-on-azure-large-instances"></a>SAP HANA en Azure (instancias grandes)

Una serie de documentos le guían a través de SAP HANA en Azure (instancias grandes) o, para abreviar, las instancias grandes de HANA. Para obtener información sobre las instancias grandes de HANA, empiece con el documento [Introducción y arquitectura de SAP HANA en Azure (instancias grandes)](./hana-overview-architecture.md) y consulte la documentación relacionada en la sección de instancias grandes de HANA



## <a name="sap-hana-on-azure-virtual-machines"></a>SAP HANA en máquinas virtuales de Azure
Esta sección de la documentación trata diferentes aspectos de SAP HANA. Antes debe estar familiarizado con los principales servicios de Azure que proporcionan servicios básicos de IaaS de Azure. Por lo tanto, necesita conocer el proceso, almacenamiento y redes de Azure. Muchos de estos temas se tratan en la [guía de planeamiento de Azure](./planning-guide.md) relacionada con SAP NetWeaver. 

 

## <a name="sap-netweaver-deployed-on-azure-virtual-machines"></a>SAP NetWeaver implementado en máquinas virtuales de Azure
En esta sección se incluye documentación sobre la planeación e implementación para SAP NetWeaver y Business One en Azure. La documentación se centra en los conceptos básicos y el uso de las bases de datos no perteneciente a HANA con una carga de trabajo de SAP en Azure. Los documentos y los artículos sobre alta disponibilidad también son la base para la alta disponibilidad de HANA en Azure como, por ejemplo:

- [Guía de plan de Azure](./planning-guide.md). 
- [SAP Business One en Azure Virtual Machines](./business-one-azure.md)
- [Protección de la implementación de una aplicación SAP NetWeaver de niveles múltiples mediante Site Recovery](../../../site-recovery/site-recovery-sap.md)
- [Conector de SAP LaMa para Azure](./lama-installation.md)

Para obtener información sobre las bases de datos no pertenecientes a HANA bajo una carga de trabajo de SAP en Azure, consulte:

- [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](./dbms_guide_general.md)
- [Implementación de DBMS de Azure Virtual Machines de SQL Server para la carga de trabajo de SAP NetWeaver](./dbms_guide_sqlserver.md)
- [Implementación de DBMS de Azure Virtual Machines de Oracle para una carga de trabajo de SAP](./dbms_guide_oracle.md)
- [Implementación de DBMS de Azure Virtual Machines de IBM DB2 para una carga de trabajo de SAP](./dbms_guide_ibm.md)
- [Implementación de DBMS de Azure Virtual Machines de SAP ASE para una carga de trabajo de SAP](./dbms_guide_sapase.md)
- [Implementación de SAP MaxDB, liveCache y del servidor de contenido en máquinas virtuales de Azure](./dbms_guide_maxdb.md)

Para obtener información sobre las bases de datos de SAP HANA en Azure, consulte la sección "SAP HANA en máquinas virtuales de Azure".

Para obtener información sobre la alta disponibilidad de una carga de trabajo de SAP en Azure, consulte:

- [Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver](./sap-high-availability-guide-start.md)

Este documento apunta a muchos otros documentos de arquitectura y escenario. En documentos posteriores del escenario, se proporcionan vínculos a documentos técnicos detallados que explican la implementación y configuración de los distintos métodos de alta disponibilidad. Los distintos documentos que muestran cómo establecer y configurar la alta disponibilidad para cargas de trabajo de SAP NetWeaver comprenden a los sistemas operativos Linux y Windows.


Para obtener información sobre la integración entre Azure Active Directory (AAD), los servicios de SAP y el inicio de sesión único, consulte:

- [Tutorial: Integración de Azure Active Directory con SAP Cloud for Customer](../../../active-directory/saas-apps/sap-customer-cloud-tutorial.md?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutorial: Integración de Azure Active Directory con SAP Cloud Platform Identity Authentication](../../../active-directory/saas-apps/sap-hana-cloud-platform-identity-authentication-tutorial.md?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutorial: Integración de Azure Active Directory con SAP Cloud Platform](../../../active-directory/saas-apps/sap-hana-cloud-platform-tutorial.md?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutorial: Integración de Azure Active Directory con SAP NetWeaver](../../../active-directory/saas-apps/sap-netweaver-tutorial.md?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutorial: Integración de Azure Active Directory con SAP Business ByDesign](../../../active-directory/saas-apps/sapbusinessbydesign-tutorial.md?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutorial: Integración de Azure Active Directory con SAP HANA](../../../active-directory/saas-apps/saphana-tutorial.md?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Your S/4HANA environment: Fiori Launchpad SAML single sign-on with Azure AD](https://blogs.sap.com/2017/02/20/your-s4hana-environment-part-7-fiori-launchpad-saml-single-sing-on-with-azure-ad/) (Entorno S/4HANA: inicio de sesión único basado en SAML para Fiori Launchpad con Azure AD)

Para obtener información sobre la integración de servicios de Azure en los componentes de SAP, consulte:

- [Uso de SAP HANA en Power BI Desktop](/power-bi/desktop-sap-hana)
- [DirectQuery y SAP HANA](/power-bi/desktop-directquery-sap-hana)
- [Uso del conector SAP BW en Power BI Desktop](/power-bi/desktop-sap-bw-connector) 
- [Azure Data Factory ofrece integración de datos de SAP HANA y Business Warehouse](https://azure.microsoft.com/blog/azure-data-factory-offer-sap-hana-and-business-warehouse-data-integration)


## <a name="change-log"></a>Registro de cambios

- 04/08/2020: Cambio en [Configuración de Pacemaker en SUSE Linux Enterprise Server en Azure](./high-availability-guide-suse-pacemaker.md) y [Configuración de Pacemaker en Red Hat Enterprise Linux en Azure](./high-availability-guide-rhel-pacemaker.md) para resaltar la importancia de la resolución de nombres confiable en los clústeres de Pacemaker.
- 04/08/2020: Cambio en [Instalación de alta disponibilidad para SAP NetWeaver en un clúster de conmutación por error de Windows y un recurso compartido de archivos para instancias de SAP ASCS/SCS](./sap-high-availability-installation-wsfc-file-share.md), [Instalación de alta disponibilidad para SAP NetWeaver en un clúster de conmutación por error de Windows y un disco compartido para una instancia de ASCS/SCS de SAP en Azure](./sap-high-availability-installation-wsfc-shared-disk.md), [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure](./high-availability-guide.md), [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications](./high-availability-guide-suse.md), [Alta disponibilidad de SAP NetWeaver en VM de Azure en SUSE Linux Enterprise Server con Azure NetApp Files para las aplicaciones de SAP](./high-availability-guide-suse-netapp-files.md), [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications: guía de varios SID](./high-availability-guide-suse-multi-sid.md), [Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver en Red Hat Enterprise Linux](./high-availability-guide-rhel.md), [Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver en Red Hat Enterprise Linux con Azure NetApp Files para aplicaciones SAP](./high-availability-guide-rhel-netapp-files.md) y [Alta disponibilidad para SAP NetWeaver en VM de Azure en Red Hat Enterprise Linux para SAP Applications: guía de varios SID](./high-availability-guide-rhel-multi-sid.md) para aclarar el uso del parámetro `enque/encni/set_so_keepalive`.
- 23/07/2020: Se ha agregado el artículo [Ahorre en SAP HANA (instancias grandes) con una reserva de Azure](../../../cost-management-billing/reservations/prepay-hana-large-instances-reserved-capacity.md), en el que se explica lo que debe saber antes de comprar una reserva SAP HANA (instancias grandes) y cómo hacer la compra.
- 16/07/2020: descripción del uso de Azure PowerShell para instalar la nueva extensión de máquina virtual para SAP en la [guía de implementación](deployment-guide.md).
- 04/7/2020: lanzamiento de [Azure Monitor para soluciones de SAP (versión preliminar)](./azure-monitor-overview.md)
- 01/07/2020: sugerencia de una configuración de almacenamiento menos costosa basada en la función de ráfaga del almacenamiento premium de Azure en el documento [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md). 
- 24/06/2020: cambio en [Configuración de Pacemaker en SLES en Azure](./high-availability-guide-suse-pacemaker.md) para lanzar una nueva versión mejorada del agente de barrera de Azure y una configuración de STONITH más resistente para dispositivos, basada en el agente de barrera de Azure. 
- 24/06/2020: cambio en [Configuración de Pacemaker en RHEL en Azure](./high-availability-guide-rhel-pacemaker.md) para lanzar una configuración de STONITH más resistente.
- 23/06/2020: cambios en la guía [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver](./planning-guide.md) y en la introducción de la guía [Tipos de Azure Storage para una carga de trabajo de SAP](./planning-guide-storage.md).
- 22/06/2020: incorporación de pasos de instalación de la nueva extensión de máquina virtual para SAP en la [guía de implementación](deployment-guide.md).
- 16/06/2020: cambio en [Conectividad del punto de conexión público para las máquinas virtuales que usan Azure Standard ILB en escenarios de alta disponibilidad de SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md) para agregar un vínculo a la documentación de la infraestructura en la nube pública 101 de SUSE. 
- 10/06/2020: incorporación de nuevas SKU de HLI en [SKU disponibles para HLI](./hana-available-skus.md) y [Arquitectura de almacenamiento de SAP HANA (instancias grandes)](./hana-storage-architecture.md).
- 21/05/2020: Cambio en [Configuración de Pacemaker en SLES en Azure](./high-availability-guide-suse-pacemaker.md) y [Configuración de Pacemaker en RHEL en Azure](./high-availability-guide-rhel-pacemaker.md) para agregar un vínculo a [Conectividad del punto de conexión público para las máquinas virtuales que usan Azure Standard ILB en escenarios de alta disponibilidad de SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md)  
- 19/05/2020: Agregar un mensaje importante para que no use el grupo de volúmenes raíz al usar LVM para volúmenes relacionados con HANA en [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md)
- 19/05/2020: incorporación de un nuevo sistema operativo compatible para el tipo de instancia grande de HANA II en [Sistemas operativos compatibles con instancias grandes de HANA](/- azure/virtual-machines/workloads/sap/os-compatibility-matrix-hana-large-instance).
- 12/05/2020: Cambio en la [Conectividad del punto de conexión público para las máquinas virtuales que usan Azure Standard ILB en escenarios de alta disponibilidad de SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md) para actualizar vínculos y agregar información para la configuración de Firewall de terceros
- 11/05/2020: Cambio en [alta disponibilidad de SAP HANA en máquinas virtuales de Azure en SLES](./sap-hana-high-availability.md) para establecer la adherencia de recursos en 0 para el recurso netcat, ya que esto conduce a una conmutación por error más simplificada 
- 05/05/2020: Cambios en [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver](./planning-guide.md) para expresar que las implementaciones de Gen2 están disponibles para la familia Mv1 VM
- 24/04/2020: cambios en [Escalabilidad horizontal de SAP HANA con el nodo en espera en máquinas virtuales de Azure con ANF en SLES](./sap-hana-scale-out-standby-netapp-files-suse.md), en [Escalabilidad horizontal de SAP HANA con el nodo en espera en máquinas virtuales de Azure con ANF en RHEL](./sap-hana-scale-out-standby-netapp-files-rhel.md), [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SLES con ANF](./high-availability-guide-suse-netapp-files.md) y [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en RHEL con ANF](./high-availability-guide-rhel-netapp-files.md) para agregar aclaración de que las direcciones IP de los volúmenes y se asignan automáticamente
- 22/04/2020: cambio en [Alta disponibilidad de SAP HANA en máquinas virtuales de Azure en SLES](./sap-hana-high-availability.md) para quitar el atributo meta `is-managed` de las instrucciones, ya que entra en conflicto con la colocación del clúster en el modo de mantenimiento o fuera de él
- 21/04/2020: se ha agregado SQL Azure DB como sistema de administración de bases de datos compatible con la plataforma de comercio de SAP (Hybris) 1811, y versiones posteriores, en los artículos [Qué software de SAP es compatible para su implementación en Azure](./sap-supported-product-on-azure.md) y [Configuraciones y certificaciones de SAP que se ejecutan en Microsoft Azure](./sap-certifications.md)
- 16/04/2020: se ha agregado SAP HANA como sistema de administración de bases de datos compatible con la plataforma de comercio de SAP (Hybris) en los artículos [Qué software de SAP es compatible para su implementación en Azure](./sap-supported-product-on-azure.md) y [Configuraciones y certificaciones de SAP que se ejecutan en Microsoft Azure](./sap-certifications.md)
- 13/04/2020: Corrección por los números de versión exactos de SAP ASE en [Implementación de DBMS de Azure Virtual Machines de SAP ASE para la carga de trabajo de SAP](./dbms_guide_sapase.md)
- 07/04/2020: Cambio en [Configuración de Pacemaker en SLES en Azure](./high-availability-guide-suse-pacemaker.md) para aclarar las instrucciones de cloud-netconfig-azure
- 06/04/2020: Cambios en [Escalabilidad horizontal de SAP HANA con nodo en espera en VM de Azure mediante Azure NetApp Files en SLES](./sap-hana-scale-out-standby-netapp-files-suse.md) y en [Escalabilidad horizontal de SAP HANA con nodo en espera en MV de Azure mediante Azure NetApp Files en RHEL](./sap-hana-scale-out-standby-netapp-files-rhel.md) para quitar referencias a [TR-4435](https://www.netapp.com/us/media/tr-4746.pdf) (sustituido por [TR-4746](https://www.netapp.com/us/media/tr-4746.pdf))
- 31/03/2020: Cambio en [Alta disponibilidad de SAP HANA en máquinas virtuales de Azure en SLES](./sap-hana-high-availability.md) y en [Alta disponibilidad de SAP HANA en máquinas virtuales de Azure en RHEL](./sap-hana-high-availability-rhel.md) para agregar instrucciones sobre cómo especificar el tamaño de sección al crear volúmenes seccionados
- 27/03/2020: Cambio en [Alta disponibilidad de SAP NetWeaver en VM de Azure en SUSE Linux Enterprise Server con Azure NetApp Files para las aplicaciones de SAP](./high-availability-guide-suse-netapp-files.md) para alinear las opciones de montaje del sistema de archivos con NetApp TR-4746 (quitar la opción de montaje de sincronización).
- 26/03/2020: Cambio en [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications: guía de varios SID](./high-availability-guide-suse-multi-sid.md) para agregar una referencia a NetApp TR-4746.
- 26/03/2020: Cambio en [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications](./high-availability-guide-suse.md), [Alta disponibilidad de SAP NetWeaver en VM de Azure en SUSE Linux Enterprise Server con Azure NetApp Files para las aplicaciones de SAP](./high-availability-guide-suse-netapp-files.md), [Alta disponibilidad para NFS en máquinas virtuales de Azure en SUSE Linux Enterprise Server](./high-availability-guide-suse-nfs.md), [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en Red Hat Enterprise Linux para SAP Applications: guía de varios SID](./high-availability-guide-suse-multi-sid.md), [Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver en Red Hat Enterprise Linux](./high-availability-guide-rhel.md) y [Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver en Red Hat Enterprise Linux con Azure NetApp Files para aplicaciones SAP](./high-availability-guide-rhel-netapp-files.md) para actualizar los diagramas y aclarar las instrucciones para la creación de grupos de back-end de Azure Load Balancer.
- 19/03/2020: Revisión principal del documento [Inicio rápido: Instalación manual de una única instancia de SAP HANA en Azure Virtual Machines](./hana-get-started.md) a [Inicio rápido: Instalación manual de una única instancia de SAP HANA en Azure Virtual Machines](./hana-get-started.md).
- 17/03/2020: Cambio en [Configuración de Pacemaker en SUSE Linux Enterprise Server en Azure](./high-availability-guide-suse-pacemaker.md) para quitar el valor de configuración de SBD que ya no es necesario.
- 16/03/2020: Aclaración del escenario de certificación de columnas en la plataforma SAP HANA con certificación de IaaS en [Qué software de SAP es compatible para su implementación en Azure](./sap-supported-product-on-azure.md).
- 11/03/2020: Cambio en [Carga de trabajo de SAP en escenarios admitidos en máquinas virtuales de Azure](./sap-planning-supported-configurations.md) para aclarar la compatibilidad de varias bases de datos por instancia de DBMS.
- 11/03/2020: Cambio en [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver](./planning-guide.md) para explicar las VM de 1.ª y 2.ª generación.
- 10/03/2020: cambio en [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md) para aclarar los límites de rendimiento existentes reales de ANF
- 09/03/2020: Cambio en [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications](./high-availability-guide-suse.md), [Alta disponibilidad de SAP NetWeaver en VM de Azure en SUSE Linux Enterprise Server con Azure NetApp Files para las aplicaciones de SAP](./high-availability-guide-suse-netapp-files.md), [Alta disponibilidad para NFS en máquinas virtuales de Azure en SUSE Linux Enterprise Server](./high-availability-guide-suse-nfs.md), [Configuración de Pacemaker en SUSE Linux Enterprise Server en Azure](./high-availability-guide-suse-pacemaker.md), [Alta disponibilidad de IBM Db2 LUW en máquinas virtuales de Azure en SUSE Linux Enterprise Server con Pacemaker](./dbms-guide-ha-ibm.md), [Alta disponibilidad de SAP HANA en máquinas virtuales de Azure en SUSE Linux Enterprise Server](./sap-hana-high-availability.md) y [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications: guía de varios SID](./high-availability-guide-suse-multi-sid.md) para actualizar los recursos de clúster con el agente de recursos azure-lb. 
- 05/03/2020: cambios de estructura y de contenido para regiones y máquinas virtuales de Azure en [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver](./planning-guide.md)
- 03/03/2020: cambio en la [alta disponibilidad para SAP NW en VM de Azure en SLES con ANF para aplicaciones de SAP](./high-availability-guide-suse-netapp-files.md) para cambiar a un diseño de volumen de ANF más eficaz.
- 01/03/2020: modificación de la [guía de copia de seguridad de SAP HANA en Azure Virtual Machines](./sap-hana-backup-guide.md) para incluir el servicio Azure Backup. Reducción y compresión del contenido de [Azure Backup de SAP HANA en el nivel de archivo](./sap-hana-backup-file-level.md), y eliminación de un tercer documento relacionado con la copia de seguridad a través de la instantánea del disco. El contenido se controla en la Guía de copia de seguridad de SAP HANA en Azure Virtual Machines 
- 27 de febrero de 2020: cambio en la [alta disponibilidad para SAP NetWeaver en VM de Azure en SLES para aplicaciones de SAP](./high-availability-guide-suse.md), la [alta disponibilidad para SAP NW en VM de Azure en SLES con ANF para aplicaciones de SAP](./high-availability-guide-suse-netapp-files.md) y la [guía de alta disponibilidad para SAP NetWeaver en VM de Azure en SLES con varios SID](./high-availability-guide-suse-multi-sid.md) para ajustar el parámetro del clúster "en caso de error".
- 26 de febrero de 2020: cambio en las [configuraciones de almacenamiento de Azure Virtual Machines de SAP HANA](./hana-vm-operations-storage.md) para aclarar la elección del sistema de archivos para HANA en Azure.
- 26 de febrero de 2020: cambio en la [arquitectura y los escenarios de alta disponibilidad para SAP](./sap-high-availability-architecture-scenarios.md) a fin de incluir el vínculo a la guía de alta disponibilidad de SAP NetWeaver en VM de Azure en RHEL con varios SID.
- 26 de febrero de 2020: cambio en la [alta disponibilidad para SAP NW en VM de Azure en SLES para aplicaciones de SAP](./high-availability-guide-suse.md), la [alta disponibilidad para SAP NW en VM de Azure en SLES con ANF para aplicaciones de SAP](./high-availability-guide-suse-netapp-files.md), la [alta disponibilidad de las VM de Azure para SAP NetWeaver en RHEL](./high-availability-guide-rhel.md) y la [alta disponibilidad de las VM de Azure para SAP NetWeaver en RHEL con Azure NetApp Files](./high-availability-guide-rhel-netapp-files.md) para quitar la declaración de incompatibilidad del clúster de ASCS/ERS con varios SID.
- 26 de febrero de 2020: publicación de la [guía de alta disponibilidad para SAP NetWeaver en VM de Azure en RHEL con varios SID](./high-availability-guide-rhel-multi-sid.md) para agregar un vínculo a la guía de clústeres de SUSE con varios SID.
- 25/02/2020: Cambio en [Escenarios y arquitectura de alta disponibilidad para SAP](./sap-high-availability-architecture-scenarios.md) para agregar vínculos a los artículos de alta disponibilidad.
- 25 de febrero de 2020: Cambio en [Alta disponibilidad de IBM Db2 LUW en VM de Azure en SUSE Linux Enterprise Server con Pacemaker](./dbms-guide-ha-ibm.md) para apuntar a un documento que describe el acceso al punto de conexión público con Azure Load Balancer estándar.
- 21 de febrero de 2020: Revisión completa del artículo [Implementación de DBMS de Azure Virtual Machines de SAP ASE para la carga de trabajo de SAP](./dbms_guide_sapase.md).
- 21 de febrero de 2020: Cambio en [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md) para representar una nueva recomendación sobre el tamaño de franja para /hana/data y agregar la configuración de un programador de E/S.
- 21 de febrero de 2020: Cambios en documentos de instancia grande de HANA para representar las SKU recién certificadas de S224 y S224m.
- 21 de febrero de 2020: Cambio en [Alta disponibilidad de VM de Azure para SAP NetWeaver en RHEL](./high-availability-guide-rhel.md) y [Alta disponibilidad de VM de Azure para SAP NetWeaver en RHEL con Azure NetApp Files](./high-availability-guide-rhel-netapp-files.md) para ajustar las restricciones del clúster a la arquitectura de replicación del servidor de puesta en cola 2 (ENSA2)
- 20 de febrero de 2020: Cambio en [Alta disponibilidad para SAP NetWeaver en VM de Azure en SLES: guía de varios SID](./high-availability-guide-suse-multi-sid.md) para agregar un vínculo a la guía de clústeres de varios SID de SUSE.
- 13 de febrero de 2020: Cambios en [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver](./planning-guide.md) para implementar vínculos a nuevos documentos.
- 13 de febrero de 2020: Incorporación de nueva documentación en [Carga de trabajo de SAP en escenarios admitidos en máquinas virtuales de Azure](./sap-planning-supported-configurations.md)
- 13 de febrero de 2020: Incorporación de nueva documentación en [¿Qué software de SAP es compatible con las implementaciones de Azure?](./sap-supported-product-on-azure.md)
- 13 de febrero de 2020: Cambio en [Alta disponibilidad de IBM Db2 LUW en VM de Azure en Red Hat Enterprise Linux Server](./high-availability-guide-rhel-ibm-db2-luw.md) para apuntar a un documento que describe el acceso al punto de conexión público con Azure Load Balancer estándar.
- 13 de febrero de 2020: Incorporación de nuevos tipos de VM en [Configuraciones y certificaciones de SAP que se ejecutan en Microsoft Azure](./sap-certifications.md).
- 13 de febrero de 2020: Incorporación de nuevas notas de compatibilidad de SAP en [Lista de comprobación de planeamiento e implementación de cargas de trabajo de SAP en Azure](./sap-deployment-checklist.md).
- 13 de febrero de 2020: Cambio en la [alta disponibilidad de las máquinas virtuales de Azure para SAP NetWeaver en RHEL](./high-availability-guide-rhel.md) y [alta disponibilidad de las máquinas virtuales de Azure para SAP NetWeaver en RHEL con Azure NetApp Files](./high-availability-guide-rhel-netapp-files.md) para alinear los tiempos de espera de los recursos de clúster con las recomendaciones de tiempo de espera de Red Hat
- 11 de febrero de 2020: Versión de la [migración de SAP HANA en instancias grandes de Azure a máquinas virtuales de Azure](./hana-large-instance-virtual-machine-migration.md)
- 7 de febrero de 2020: Cambio en [Conectividad del punto de conexión público para las máquinas virtuales que usan Azure Standard ILB en escenarios de alta disponibilidad de SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md) para actualizar la captura de pantalla de ejemplo de NSG
- 3 de febrero de 2020: Cambio en [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications](./high-availability-guide-suse.md) y [Alta disponibilidad de SAP NetWeaver en VM de Azure en SUSE Linux Enterprise Server con Azure NetApp Files para las aplicaciones de SAP](./high-availability-guide-suse-netapp-files.md) para quitar la advertencia sobre el uso de guiones en los nombres de host de los nodos de clúster en SUSE Linux Enterprise Server.
- 28 de enero de 2020: Cambio en la [alta disponibilidad de SAP HANA en máquinas virtuales de Azure en RHEL](./sap-hana-high-availability-rhel.md) para alinear los tiempos de espera de los recursos de clúster de SAP HANA con las recomendaciones de tiempo de espera de Red Hat.
- 17 de enero de 2020: Cambio en los [grupos de selección de ubicación de proximidad de Azure para conseguir una latencia de red óptima con aplicaciones SAP](./sap-proximity-placement-scenarios.md) para cambiar la sección del movimiento de las máquinas virtuales existentes a un grupo de selección de ubicación de proximidad.
- 17 de enero de 2020: Cambio en las [configuraciones de carga de trabajo de SAP con Azure Availability Zones](./sap-ha-availability-zones.md) para que apunten al procedimiento que automatiza las mediciones de latencia entre zonas de disponibilidad
- 16 de enero de 2020: Cambio en la [instalación y configuración de SAP HANA (instancias grandes) en Azure](./hana-installation.md) para adaptar las versiones del sistema operativo al directorio de hardware de IaaS de HANA
- 16 de enero de 2020: Cambios en la [guía de varios SID de alta disponibilidad de las máquinas virtuales de Azure para SAP NetWeaver en SLES](./high-availability-guide-suse-multi-sid.md) para agregar instrucciones para sistemas SAP con la arquitectura de puesta en cola del servidor 2 (ENSA2)
- 10 de enero de 2020: Cambios en la [escalabilidad horizontal de SAP HANA con nodo en espera en máquinas virtuales de Azure con Azure NetApp Files en SLES](./sap-hana-scale-out-standby-netapp-files-suse.md) y en la [escalabilidad horizontal de SAP HANA con nodo en espera en máquinas virtuales de Azure con Azure NetApp Files en RHEL](./sap-hana-scale-out-standby-netapp-files-rhel.md) para agregar instrucciones sobre cómo hacer que los cambios de `nfs4_disable_idmapping` sean permanentes.
- 10 de enero de 2020: Cambios en [Alta disponibilidad de SAP NetWeaver en VM de Azure en SUSE Linux Enterprise Server con Azure NetApp Files para las aplicaciones de SAP](./high-availability-guide-suse-netapp-files.md) y en [Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver en Red Hat Enterprise Linux con Azure NetApp Files para aplicaciones SAP](./high-availability-guide-rhel-netapp-files.md) para agregar instrucciones sobre cómo montar volúmenes NFSv4 de Azure NetApp Files.
- 23 de diciembre de 2019: Publicación de la [guía de varios SID de alta disponibilidad de las máquinas virtuales de Azure para SAP NetWeaver en SLES](./high-availability-guide-suse-multi-sid.md)
- 18 de diciembre de 2019: Publicación de la [escalabilidad horizontal de SAP HANA con nodo en espera en máquinas virtuales de Azure con Azure NetApp Files en RHEL](./sap-hana-scale-out-standby-netapp-files-rhel.md)
- 21 de noviembre de 2019: Los cambios en [SAP HANA se escalan horizontalmente con el nodo en espera en máquinas virtuales de Azure con Azure NetApp Files en SUSE Linux Enterprise Server](./sap-hana-scale-out-standby-netapp-files-suse.md) para simplificar la configuración de la asignación de identificador de NFS y cambiar la interfaz de red principal recomendada para simplificar el enrutamiento.
- 15 de noviembre de 2019: Cambios menores en [Alta disponibilidad de SAP NetWeaver en SUSE Linux Enterprise Server con Azure NetApp Files para aplicaciones de SAP](high-availability-guide-suse-netapp-files.md) y [Alta disponibilidad para SAP NetWeaver en Red Hat Enterprise Linux con Azure NetApp Files para aplicaciones SAP](high-availability-guide-rhel-netapp-files.md) para aclarar las restricciones de tamaño del grupo de capacidad y quitar la declaración de que solo se admite la versión de NFSv3.
- 12 de noviembre de 2019: Publicación de [Alta disponibilidad de SAP NetWeaver en Windows con Azure NetApp Files (SMB)](high-availability-guide-windows-netapp-files-smb.md)
- 8 de noviembre de 2019: Cambios en [Alta disponibilidad de SAP HANA en máquinas virtuales de Azure en SUSE Linux Enterprise Server](sap-hana-high-availability.md), [Configuración de la replicación del sistema SAP HANA en máquinas virtuales de Azure](sap-hana-high-availability-rhel.md), [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications](high-availability-guide-suse.md), [Alta disponibilidad de SAP NetWeaver en VM de Azure en SUSE Linux Enterprise Server con Azure NetApp Files para las aplicaciones de SAP](high-availability-guide-suse-netapp-files.md), [Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver en Red Hat Enterprise Linux](high-availability-guide-rhel.md), [Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver en Red Hat Enterprise Linux con Azure NetApp Files para aplicaciones SAP](high-availability-guide-rhel-netapp-files.md), [Alta disponibilidad para NFS en máquinas virtuales de Azure en SUSE Linux Enterprise Server](high-availability-guide-suse-nfs.md), [GlusterFS en máquinas virtuales de Azure de Red Hat Enterprise Linux para SAP NetWeaver](high-availability-guide-rhel-glusterfs.md) para recomendar el equilibrador de carga estándar de Azure  
- 8 de noviembre de 2019: Cambios en [Lista de comprobación de planeamiento e implementación de cargas de trabajo de SAP en Azure](sap-deployment-checklist.md) para aclarar la recomendación de cifrado  
- 4 de noviembre de 2019: Cambios en [Configuración de Pacemaker en SUSE Linux Enterprise Server en Azure](high-availability-guide-suse-pacemaker.md) para crear el clúster directamente con la configuración de unidifusión.  
