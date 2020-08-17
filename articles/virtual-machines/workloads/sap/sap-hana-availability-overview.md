---
title: 'Disponibilidad de SAP HANA en máquinas virtuales de Azure: Introducción | Microsoft Docs'
description: Describe las operaciones de SAP HANA en máquinas virtuales nativas de Azure.
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
ms.date: 03/05/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: e44edff9ac81588e4f861b7a1d1ae1a1628d5e04
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87035645"
---
# <a name="sap-hana-high-availability-for-azure-virtual-machines"></a>Alta disponibilidad de SAP HANA para máquinas virtuales de Azure

Puede usar numerosas funcionalidades de Azure que permiten implementar bases de datos críticas como SAP HANA en máquinas virtuales de Azure. Este artículo proporciona una guía sobre cómo lograr disponibilidad para instancias de SAP HANA que se hospedan en máquinas virtuales de Azure. En el artículo, se describen varios escenarios que se pueden implementar utilizando la infraestructura de Azure para aumentar la disponibilidad de SAP HANA en Azure. 

## <a name="prerequisites"></a>Prerequisites

En este artículo se da por supuesto que está familiarizado con los conceptos básicos de la infraestructura como servicio (IaaS) en Azure, por ejemplo: 

- Cómo implementar máquinas virtuales o redes virtuales mediante Azure Portal o PowerShell.
- Cómo usar la interfaz de línea de comandos (CLI) multiplataforma de Azure, incluida la opción para usar plantillas de notificación de objetos JavaScript (JSON).

También se da por supuesto que está familiarizado con la instalación de instancias de SAP HANA, así como con su administración y funcionamiento. Es especialmente importante que esté familiarizado con la configuración y las operaciones de replicación del sistema de HANA. Esto incluye tareas como la copia de seguridad y restauración de bases de datos de SAP HANA.

Estos artículos proporcionan una buena descripción general del uso de SAP HANA en Azure:

- [Inicio rápido: instalación manual de una única instancia de SAP HANA en máquinas virtuales de Azure](./hana-get-started.md)
- [Configuración de la replicación del sistema de SAP HANA en máquinas virtuales de Azure](sap-hana-high-availability.md)
- [Copia de seguridad de SAP HANA en máquinas virtuales de Azure](./sap-hana-backup-guide.md)

También es conveniente estar familiarizado con estos artículos acerca de SAP HANA:

- [High Availability for SAP HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.02/en-US/6d252db7cdd044d19ad85b46e6c294a4.html) (Alta disponibilidad para SAP HANA)
- [FAQ: High Availability for SAP HANA](https://archive.sap.com/documents/docs/DOC-66702) (Preguntas más frecuentes: alta disponibilidad para SAP HANA)
- [Perform system replication for SAP HANA](https://archive.sap.com/documents/docs/DOC-47702) (Cómo realizar la replicación del sistema de SAP HANA)
- [SAP HANA 2.0 SPS 01 What’s New: High Availability](https://blogs.sap.com/2017/05/15/sap-hana-2.0-sps-01-whats-new-high-availability-by-the-sap-hana-academy/) (Novedades de SAP HANA 2.0 SPS 01: alta disponibilidad)
- [Network Recommendations for SAP HANA System Replication](https://www.sap.com/documents/2016/06/18079a1c-767c-0010-82c7-eda71af511fa.html) (Recomendaciones de red para la replicación del sistema de SAP HANA)
- [SAP HANA System Replication](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/b74e16a9e09541749a745f41246a065e.html) (Replicación del sistema de SAP HANA)
- [SAP HANA Service Auto-Restart](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/cf10efba8bea4e81b1dc1907ecc652d3.html) (Servicio de reinicio automático de SAP HANA)
- [Configure SAP HANA system replication](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/676844172c2442f0bf6c8b080db05ae7.html) (Configuración de la replicación del sistema de SAP HANA)

Más allá de estar familiarizado con la implementación de máquinas virtuales en Azure, también se recomienda leer el artículo [Administración de la disponibilidad de las máquinas virtuales Windows en Azure](../../windows/manage-availability.md) antes de continuar con la definición de la arquitectura de disponibilidad en Azure.

## <a name="service-level-agreements-for-azure-components"></a>Acuerdos de Nivel de Servicio para los componentes de Azure

Azure tiene diferentes Acuerdos de Nivel de Servicio de disponibilidad para los distintos componentes como redes, almacenamiento y máquinas virtuales. Todos los Acuerdos de Nivel de Servicio están documentados. Para más información, consulte [Acuerdos de Nivel de Servicio de Microsoft Azure](https://azure.microsoft.com/support/legal/sla/). 

[Acuerdo de Nivel de Servicio para Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/) describe tres Acuerdos de Nivel de Servicio diferentes, para tres configuraciones distintas:

- Una sola máquina virtual que usa [discos SSD Premium de Azure ](../../windows/managed-disks-overview.md) para el disco del sistema operativo y todos los discos de datos. Esta opción proporciona un tiempo de actividad mensual del 99,9 %.
- Varias máquinas virtuales (al menos dos) que se organizan en un [conjunto de disponibilidad de Azure](../../windows/tutorial-availability-sets.md). Esta opción proporciona un tiempo de actividad mensual del 99,95 %.
- Varias máquinas virtuales (al menos dos) que se organizan en una [zona de disponibilidad](../../../availability-zones/az-overview.md). Esta opción proporcionaba un tiempo de actividad mensual del 99,99 %.

Mida el requisito de disponibilidad en los Acuerdos de Nivel de Servicio que los componentes de Azure pueden proporcionar. A continuación, elija los escenarios de SAP HANA para lograr el nivel requerido de disponibilidad.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [disponibilidad de SAP HANA dentro de una región de Azure](./sap-hana-availability-one-region.md).
- Más información sobre [disponibilidad de SAP HANA entre regiones de Azure](./sap-hana-availability-across-regions.md). 















  
