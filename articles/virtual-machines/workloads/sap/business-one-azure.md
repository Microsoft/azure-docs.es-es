---
title: SAP Business One en Azure Virtual Machines | Microsoft Docs
description: SAP Business One en Azure.
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
ms.date: 07/15/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: ccec58f012dcd4b6371c15e79fa964600e775f54
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654657"
---
# <a name="sap-business-one-on-azure-virtual-machines"></a>SAP Business One en Azure Virtual Machines
En este documento se ofrecen instrucciones para implementar SAP Business One en Azure Virtual Machines. La documentación no es un sustituto de la documentación de instalación de Business One para SAP. La documentación debe cubrir las instrucciones básicas de planeación e implementación de la infraestructura de Azure en la que se ejecutan las aplicaciones de Business One.

Business One admite dos bases de datos diferentes:
- SQL Server: vea la [Nota de SAP 928839: Release Planning for Microsoft SQL Server](https://launchpad.support.sap.com/#/notes/928839) (Planeamiento de versiones para Microsoft SQL Server).
- SAP HANA: para obtener una matriz de compatibilidad de SAP Business One exacta para SAP HANA, consulte la [matriz de disponibilidad de productos de SAP](https://support.sap.com/pam).

Con respecto a SQL Server, se aplican las consideraciones de implementación básicas documentadas en [Implementación de DBMS de Azure Virtual Machines para SAP NetWeaver](./dbms_guide_general.md). Para SAP HANA, las consideraciones se mencionan en este documento.

## <a name="prerequisites"></a>Requisitos previos
Para utilizar esta guía, necesita un conocimiento básico de los siguientes componentes de Azure:

- [Máquinas virtuales de Azure en Windows](../../windows/tutorial-manage-vm.md)
- [Máquinas virtuales de Azure en Linux](../../linux/tutorial-manage-vm.md)
- [Redes de Azure y administración de redes virtuales con PowerShell](../../windows/tutorial-virtual-network.md)
- [Redes de Azure y redes virtuales con la CLI](../../linux/tutorial-virtual-network.md)
- [Administración de discos de Azure con la CLI de Azure](../../linux/tutorial-manage-disks.md)

Incluso si solo está interesado en Business One, el documento [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver](./planning-guide.md) puede ser una buena fuente de información.

Se supone que como la instancia que va a implementar SAP Business One:

- Está familiarizado con la instalación de SAP HANA en una infraestructura dada como una máquina virtual.
- Está familiarizado con la instalación de la aplicación SAP Business One en una infraestructura como máquinas virtuales de Azure.
- Está familiarizado con SAP Business One y el sistema DBMS elegido.
- Está familiarizado con la implementación de la infraestructura en Azure.

Todas estas áreas no se tratarán en este documento.

Además de la documentación de Azure, debe tener en cuenta las notas de SAP principales, que hacen referencia a Business One o que son notas centrales de SAP para Business One:

- [528296: Nota de información general para versiones y productos relacionados de SAP Business One](https://launchpad.support.sap.com/#/notes/528296)
- [2216195: Nota de actualizaciones de versión para SAP Business One 9.2, versión para SAP HANA](https://launchpad.support.sap.com/#/notes/2216195)
- [2483583: Nota central para SAP Business One 9.3](https://launchpad.support.sap.com/#/notes/2483583)
- [2483615: Nota de actualizaciones de versión para SAP Business One 9.3](https://launchpad.support.sap.com/#/notes/2483615)
- [2483595: Nota colectiva para problemas generales de SAP Business One 9.3](https://launchpad.support.sap.com/#/notes/2483595)
- [2027458: Nota de consultoría colectiva para temas relacionados con SAP HANA de SAP Business One, versión para SAP HANA](https://launchpad.support.sap.com/#/notes/2027458)


## <a name="business-one-architecture"></a>Arquitectura de Business One
Business One es una aplicación que tiene dos niveles:

- Un nivel de cliente con un cliente "pesado".
- Un nivel de base de datos que contiene el esquema de base de datos para un inquilino.

Una mejor información general sobre qué componentes se ejecutan en la parte del cliente y cuáles se ejecutan en la parte del servidor se documenta en la [guía del administrador de SAP Business One](https://help.sap.com/doc/601fbd9113be4240b81d74626439cfa9/10.0/en-US/AdministratorGuide_SQL.pdf). 

Puesto que hay una interacción crítica de latencia elevada entre el nivel de cliente y el de DBMS, ambos niveles deben estar ubicados en Azure cuando se implementa en Azure. Es habitual que los usuarios utilicen RDS en una o varias máquinas virtuales que ejecutan un servicio RDS para los componentes de cliente de Business One.

### <a name="sizing-vms-for-sap-business-one"></a>Ajuste del tamaño de las máquinas virtuales para SAP Business One

Con respecto al tamaño de las máquinas virtuales cliente, los requisitos de recursos están documentados por SAP en el documento [SAP Business One Hardware Requirements Guide](https://help.sap.com/doc/bfa9770d12284cce8509956dcd4c5fcb/9.3/en-US/B1_Hardware_Requirements_Guide.pdf) (Guía de requisitos de hardware de SAP Business One). Para Azure, necesita centrarse y calcular los requisitos indicados en el capítulo 2.4 del documento.

Como máquinas virtuales de Azure para hospedar los componentes de cliente de Business One y el host DBMS, solo se permiten máquinas virtuales compatibles con SAP NetWeaver. Para obtener la lista de máquinas virtuales de Azure compatibles con SAP NetWeaver, lea la [nota de SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533).

Al ejecutar SAP HANA como back-end de DBMS para Business One, solo se admiten para HANA las máquinas virtuales que se enumeran para Business One HANA en la [lista de plataformas de IaaS certificadas para HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure%23SAP%20Business%20One). Los componentes de cliente de Business One no se ven afectados por esta restricción más estricta para SAP HANA como sistema DBMS.

### <a name="operating-system-releases-to-use-for-sap-business-one"></a>Versiones de sistema operativo para SAP Business One

En principio, siempre es mejor usar las versiones más recientes del sistema operativo. Especialmente en el espacio de Linux, se introdujeron nuevas funciones de Azure con diferentes versiones secundarias más recientes de Suse y Red Hat. En Windows, se recomienda encarecidamente usar Windows Server 2016.


## <a name="deploying-infrastructure-in-azure-for-sap-business-one"></a>Implementación de infraestructura en Azure para SAP Business One
En los capítulos siguientes, se muestran los componentes de infraestructura relevantes para la implementación de SAP.

### <a name="azure-network-infrastructure"></a>Infraestructura de red de Azure
La infraestructura de red que necesita para implementar en Azure depende de si implementa un único sistema de Business One para usted mismo. O bien, si es un proveedor de hospedaje que hospeda docenas de sistemas de Business One para los clientes. Es posible que también haya pequeños cambios en el diseño en función de cómo se conecte a Azure. Pasando por las diferentes posibilidades, un diseño donde haya conectividad VPN a Azure y donde se amplíe Active Directory a través de [VPN](../../../vpn-gateway/vpn-gateway-about-vpngateways.md) o [ExpressRoute](../../../expressroute/expressroute-introduction.md) en Azure.

![Configuración de red simple con Business One](./media/business-one-azure/simple-network-with-VPN.PNG)

En la configuración simplificada que se describe se presentan varias instancias de seguridad que permiten controlar y limitar el enrutamiento. Comienza con 

- El enrutador o firewall en el lado local del cliente.
- La siguiente instancia es el [Grupo de seguridad de red de Azure](../../../virtual-network/security-overview.md), que se puede usar para agregar las reglas de enrutamiento y seguridad para la red virtual de Azure en la que se ejecuta la configuración de SAP Business One.
- Para evitar que los usuarios del cliente de Business One también puedan ver el servidor que ejecuta el servidor de Business One, que ejecuta la base de datos, debe separar la máquina virtual en la que se hospeda el cliente de Business One y el servidor de Business One en dos subredes diferentes dentro de la red virtual.
- Se volvería a usar el grupo de seguridad de Azure asignado a las dos subredes para limitar el acceso al servidor de Business One.

Una versión más sofisticada de una configuración de red de Azure se basa en los [procedimientos recomendados documentados de la arquitectura de tipo hub-and-spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) de Azure. El patrón de arquitectura de tipo hub-and-spoke cambiaría la primera configuración simplificada por una similar a la siguiente:


![Configuración de tipo hub-and-spoke con Business One](./media/business-one-azure/hub-spoke-network-with-VPN.PNG)

Para los casos donde los usuarios se conectan a través de Internet sin conectividad privada a Azure, el diseño de la red de Azure se debe alinear con los principios que se documentan en la referencia de arquitectura de Azure para [Red perimetral entre Internet y Azure](/azure/architecture/reference-architectures/dmz/secure-vnet-dmz).

### <a name="business-one-database-server"></a>Servidor de base de datos de Business One
Para el tipo de base de datos, están disponibles SQL Server y SAP HANA. Con independencia del DBMS, debe leer el documento [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](./dbms_guide_general.md) a fin de obtener una descripción general de las implementaciones de DBMS en máquinas virtuales de Azure y los temas relacionados de redes y almacenamiento.

Aunque ya se destaca en los documentos de base de datos genéricos y específicos, debería estar familiarizado con lo siguiente:

- [Administración de la disponibilidad de las máquinas virtuales Windows en Azure](../../windows/manage-availability.md) y [Administración de la disponibilidad de máquinas virtuales Linux en Azure](../../linux/manage-availability.md)
- [Contrato de nivel de servicio para Máquinas virtuales](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/)

Estos documentos le ayudarán a decidir la selección de los tipos de almacenamiento y la configuración de alta disponibilidad.

En principio, debería:

- Usar discos SSD Premium a través de unidades de disco duro estándar. Para más información acerca de los tipos de disco disponibles, consulte nuestro artículo [Selección de un tipo de disco](../../disks-types.md).
- Usar Azure Managed Disks antes que discos no administrados.
- Asegurarse de que tiene suficiente rendimiento de IOPS y E/S configurado con la configuración de discos.
- Combinar el volumen /hana/data y /hana/log para disponer de una configuración de almacenamiento rentable.


#### <a name="sql-server-as-dbms"></a>SQL Server como DBMS
Para implementar SQL Server como DBMS para Business One, vea el documento [Implementación de DBMS de SQL Server Azure Virtual Machines para SAP NetWeaver](./dbms_guide_sqlserver.md). 

Las estimaciones de tamaño aproximadas para el lado DBMS de SQL Server son las siguientes:

| Número de usuarios | vCPU | Memoria | Tipos de máquina virtual de ejemplo |
| --- | --- | --- | --- |
| Hasta 20 | 4 | 16 GB | D4s_v3, E4s_v3 |
| Hasta 40 | 8 | 32 GB | D8s_v3, E8s_v3 |
| Hasta 80 | 16 | 64 GB | D16s_v3, E16s_v3 |
| Hasta 150 | 32 | 128 GB | D32s_v3, E32s_v3 |

El ajuste de tamaño indicado anteriormente debería darle una idea de por dónde empezar. Es posible que necesite más o menos recursos, en cuyo caso la adaptación en Azure es fácil. Se pueden cambiar los tipos de máquina virtual con un simple reinicio de la máquina virtual.

#### <a name="sap-hana-as-dbms"></a>SAP HANA como DBMS
Si se usa SAP HANA como DBMS en las secciones siguientes, se deben seguir las consideraciones del documento [Guía de operaciones de SAP HANA en Azure](./hana-vm-operations.md).

Para las configuraciones de alta disponibilidad y recuperación ante desastres de SAP HANA como base de datos de Business One en Azure, debe leer la documentación de [Alta disponibilidad de SAP HANA para máquinas virtuales de Azure](./sap-hana-availability-overview.md) y la documentación a la que se señala desde ese documento.

Para las estrategias de copia de seguridad y restauración de SAP HANA, debería leer el documento [Guía de copia de seguridad de SAP HANA en Azure Virtual Machines](./sap-hana-backup-guide.md) y la documentación a la que se señala desde ese documento.

 
### <a name="business-one-client-server"></a>Servidor del cliente de Business One
Para estos componentes, las consideraciones sobre el almacenamiento no son la principal preocupación. Pero le interesa contar con una plataforma confiable. Por tanto, debe usar Azure Premium Storage para esta máquina virtual, incluso para el disco duro virtual base. Ajuste el tamaño de la máquina virtual, con los datos proporcionados en la [guía de requisitos de hardware de SAP Business One](https://help.sap.com/http.svc/rc/011000358700000244612011e/9.3/en-US/B1_Hardware_Requirements_Guide.pdf). Para Azure, necesita centrarse y calcular los requisitos indicados en el capítulo 2.4 del documento. Mientras calcula los requisitos, debe compararlos con los documentos siguientes para encontrar la máquina virtual ideal:

- [Tamaños de las máquinas virtuales Windows en Azure](../../sizes.md)
- [Nota de SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533)

Compare el número de CPU y memoria necesaria con lo que Microsoft documenta. Tenga en cuenta también el rendimiento de la red al elegir las máquinas virtuales.