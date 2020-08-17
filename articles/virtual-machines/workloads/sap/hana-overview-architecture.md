---
title: Introducción a SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: Introducción a la implementación de SAP HANA en Azure (instancias grandes).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/12/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: e1575ee02a520fd39a1c659eadaeda5822207539
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87171133"
---
#  <a name="what-is-sap-hana-on-azure-large-instances"></a>¿Qué es SAP HANA en Azure (instancias grandes)?

SAP HANA en Azure (instancias grandes) es una solución exclusiva de Azure. Además de proporcionar máquinas virtuales con el fin de implementar y ejecutar SAP HANA, Azure ofrece la posibilidad de ejecutar e implementar SAP HANA en servidores sin sistema operativo dedicados a su organización. La solución SAP HANA en Azure (instancias grandes) se basa en hardware sin sistema operativo con hosts o servidores no compartidos asignados a su organización. El hardware del servidor se inserta en las marcas más grandes que contienen infraestructuras de proceso/servidor, redes y almacenamiento. De forma combinada, tiene la certificación de integración de centro de datos adaptado para HANA (TDI). SAP HANA en Azure (instancias grandes) ofrece diferentes SKU o tamaños de servidor. Las unidades pueden tener 36 núcleos de CPU Intel y 768 GB de memoria y aumentar hasta unidades que tengan 480 núcleos de CPU Intel y 24 TB de memoria como máximo.

El aislamiento del cliente dentro de la demarcación de infraestructura se realiza en inquilinos y presenta el siguiente aspecto:

- **Redes**: aislamiento de los clientes dentro de la pila de infraestructura mediante redes virtuales por inquilino asignado al cliente. Un inquilino se asigna a un único cliente. Un cliente puede tener varios inquilinos. El aislamiento de red de los inquilinos prohíbe la comunicación de red entre ellos en el nivel de la demarcación de infraestructura, incluso si los inquilinos pertenecen al mismo cliente.
- **Componentes de almacenamiento**: aislamiento mediante máquinas virtuales de almacenamiento que tienen asignados volúmenes de almacenamiento. Los volúmenes de almacenamiento pueden asignarse solo a una máquina virtual de almacenamiento. Una máquina virtual de almacenamiento se asigna exclusivamente a un solo inquilino en la pila de la infraestructura certificada con SAP HANA TDI. Como resultado, se puede acceder a los volúmenes de almacenamiento asignados a una máquina virtual de almacenamiento solo en un inquilino específico y relacionado. Dichos volúmenes no son visibles entre los distintos inquilinos implementados.
- **Servidor o host**: una unidad de servidor o host no se comparte entre clientes o inquilinos. Un servidor o host implementado en un cliente es una unidad de proceso de reconstrucción completa atómica que se asigna a un único inquilino. *No* se usan particiones por hardware ni por software que puedan dar lugar a que un cliente comparta un host o servidor con otro cliente. Los volúmenes de almacenamiento asignados a la máquina virtual de almacenamiento del inquilino específico se montan en dicho servidor. Un inquilino puede tener una o muchas unidades de servidor de diferentes SKU asignadas de forma exclusiva.
- En una demarcación de infraestructura de SAP HANA en Azure (instancias grandes) se implementan muchos inquilinos diferentes que se aíslan entre sí a través de los conceptos de inquilino en los niveles de redes, almacenamiento y proceso. 


La ejecución de estas unidades de servidor de reconstrucción completa es compatible solo en SAP HANA. El nivel de aplicación de SAP o el nivel de middleware de la carga de trabajo se ejecutan en máquinas virtuales. Las demarcaciones de infraestructura que ejecutan las unidades de SAP HANA en Azure (instancias grandes) están conectadas a las redes troncales de los servicios de red de Azure. De este modo, se proporciona conectividad de baja latencia entre las unidades de SAP HANA en Azure (instancias grandes) y las máquinas virtuales.

A partir de julio de 2019, diferenciamos entre dos revisiones distintas de los sellos de HANA (instancias grandes) y la ubicación de las implementaciones:

- "Revisión 3" (Rev 3): se trata de los sellos que se pusieron a disposición del cliente para su implementación antes de julio de 2019
- "Revisión 4" (Rev 4): nuevo diseño de sello que se implementa cerca de los hosts de máquina virtual de Azure y que hasta ahora se publicaban en las siguiente regiones:
    -  Oeste de EE. UU. 2 
    -  Este de EE. UU. 
    -  Oeste de Europa
    -  Norte de Europa


Este documento es uno de varios documentos que tratan sobre SAP HANA en Azure (instancias grandes). Este documento presenta la arquitectura básica, las responsabilidades y los servicios proporcionados por la solución. También se describen las funcionalidades de alto nivel de la solución. Para la mayoría de otras áreas, como redes y conectividad, otros cuatro documentos tratan detalles e información en profundidad. La documentación de SAP HANA en Azure (instancias grandes) no aborda aspectos de la instalación de SAP NetWeaver ni de las implementaciones de SAP NetWeaver en máquinas virtuales. SAP NetWeaver en Azure se trata en documentos independientes disponibles en el mismo contenedor de documentación de Azure. 


Los distintos documentos de orientación de la instancia grande de HANA abarcan las siguientes áreas:

- [Introducción y arquitectura de SAP HANA en Azure (instancias grandes)](hana-overview-architecture.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Infraestructura y conectividad con SAP HANA en Azure (instancias grandes)](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Instalación y configuración de SAP HANA en Azure (instancias grandes)](hana-installation.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Alta disponibilidad y recuperación ante desastres de SAP HANA en Azure (instancias grandes)](hana-overview-high-availability-disaster-recovery.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Solución de problemas y supervisión de SAP HANA en Azure (instancias grandes)](troubleshooting-monitoring.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Configuración de alta disponibilidad en SUSE mediante STONITH](./ha-setup-with-stonith.md)
- [Copia de seguridad y restauración del sistema operativo para SKU de tipo II de los sellos de revisión 3](./os-backup-type-ii-skus.md)
- [Ahorre en SAP HANA (instancias grandes) con una reserva de Azure](../../../cost-management-billing/reservations/prepay-hana-large-instances-reserved-capacity.md)

**Pasos siguientes**
- Consulte [Conocer los términos](hana-know-terms.md).
