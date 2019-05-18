---
title: Certificaciones de Microsoft Azure para SAP | Microsoft Docs
description: Lista actualizada de las configuraciones y certificaciones actuales de SAP en la plataforma Azure.
services: virtual-machines-linux
documentationcenter: ''
author: RicksterCDN
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 05/15/2019
ms.author: rclaus
ms.custom: ''
ms.openlocfilehash: a38d5b44aaca0aa28e7ecd32543d2b0563964b3d
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65797442"
---
# <a name="sap-certifications-and-configurations-running-on-microsoft-azure"></a>Configuraciones y certificaciones de SAP que se ejecutan en Microsoft Azure

SAP y Microsoft tienen una larga historia de trabajo conjunto en una asociación segura que tiene ventajas mutuas para sus clientes. Microsoft actualiza constantemente su plataforma y envía nuevos detalles de certificación a SAP, con el fin de asegurarse de que Microsoft Azure es la mejor plataforma para ejecutar las cargas de trabajo de SAP. Las tablas siguientes describen nuestras configuraciones admitidas de Azure y una lista de certificaciones SAP en crecimiento. 

## <a name="sap-hana-certifications"></a>Certificaciones SAP HANA
Referencias:

- [SAP HANA certified IaaS platforms](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) (Plataformas IaaS con certificación SAP HANA) para soporte técnico para máquinas virtuales de Azure nativas e instancias grandes de HANA.

| Producto de SAP | SO admitido | Productos de Azure |
| --- | --- | --- |
| SAP HANA Developer Edition (incluido el software cliente de HANA compuesto por los controladores SQLODBC, ODBO solo para Windows, ODBC y JDBC, HANA Studio y la base de datos HANA) | Red Hat Enterprise Linux, SUSE Linux Enterprise | Familia de máquina virtual de la serie D |
| Business One on HANA | SUSE Linux Enterprise | DS14_v2, M32ts, M32ls, M64ls, M64s <br /> [Plataformas IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure%23SAP%20Business%20One) |
| SAP S/4 HANA | Red Hat Enterprise Linux, SUSE Linux Enterprise | Disponibilidad controlada para GS5. Compatibilidad total con M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, <br /> SAP HANA en Azure (instancias grandes) [Plataformas IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) |
| Suite en Hana, OLTP | Red Hat Enterprise Linux, SUSE Linux Enterprise | M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, SAP HANA en Azure (instancias grandes) <br /> [Plataformas IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) |
| Hana Enterprise para BW, OLAP | Red Hat Enterprise Linux, SUSE Linux Enterprise | GS5, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, <br /> SAP HANA en Azure (instancias grandes) [Plataformas IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) |
| SAP BW/4 HANA | Red Hat Enterprise Linux, SUSE Linux Enterprise | GS5, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, <br /> SAP Hana en Azure (instancias grandes) <br /> [Plataformas IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) |

Tenga en cuenta que SAP utiliza el término "agrupación en clústeres" en las [Plataformas IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) como sinónimo para "escalabilidad horizontal" y NO para "agrupación en clústeres" de alta disponibilidad

## <a name="sap-netweaver-certifications"></a>Certificaciones de SAP NetWeaver
Microsoft Azure cuenta con certificación para los siguientes productos de SAP, con soporte técnico completo de Microsoft y SAP.
Referencias:

- [1928533 - aplicaciones de SAP en Azure: Productos admitidos y tipos de máquinas virtuales de Azure](https://launchpad.support.sap.com/#/notes/1928533) para todas las aplicaciones basadas en SAP NetWeaver, incluidas SAP TREX, SAP LiveCache y SAP Content Server. Y todas las bases de datos, excluida SAP HANA.


| Producto de SAP | SO invitado | RDBMS | Tipos de máquinas virtuales |
| --- | --- | --- | --- |
| Software SAP Business Suite | Windows, SUSE Linux Enterprise, Red Hat Enterprise Linux, Oracle Linux |SQL Server, Oracle (solo Windows y Oracle Linux), DB2, SAP ASE |A5 a A11, D11 a D14, DS11 a DS14, DS11_v2 a DS15_v2, GS1 a GS5, D2s_v3 a D64s_v3, E2s_v3 a E64s_v3, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2 |
| SAP Business All-in-One | Windows, SUSE Linux Enterprise, Red Hat Enterprise Linux, Oracle Linux |SQL Server, Oracle (solo Windows y Oracle Linux), DB2, SAP ASE |A5 a A11, D11 a D14, DS11 a DS14, DS11_v2 a DS15_v2, GS1 a GS5, D2s_v3 a D64s_v3, E2s_v3 a E64s_v3, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2 |
| SAP BusinessObjects BI |  Windows |N/D |A5 a A11, D11 a D14, DS11 a DS14, DS11_v2 a DS15_v2, GS1 a GS5, D2s_v3 a D64s_v3, E2s_v3 a E64s_v3, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2 |
| SAP NetWeaver | Windows, SUSE Linux Enterprise, Red Hat Enterprise Linux, Oracle Linux |SQL Server, Oracle (solo Windows y Oracle Linux), DB2, SAP ASE |A5 a A11, D11 a D14, DS11 a DS14, DS11_v2 a DS15_v2, GS1 a GS5, D2s_v3 a D64s_v3, E2s_v3 a E64s_v3, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2 |

## <a name="other-sap-workload-supported-on-azure"></a>Otras cargas de trabajo de SAP admitidas en Azure

| Producto de SAP | SO invitado | RDBMS | Tipos de máquinas virtuales |
| --- | --- | --- | --- |
| SAP Business One on SQL Server |  Windows  | SQL Server | Todos los tipos de máquina virtual con certificado NetWeaver<br /> [Nota de SAP #928839](https://launchpad.support.sap.com/#/notes/928839) |
| SAP BPC 10.01 MS SP08 | Windows y Linux | | Todos los tipos de máquina virtual con certificado NetWeaver<br /> Nota de SAP 2451795 |
| SAP Business Objects BI platform | Windows y Linux | | Nota de SAP 2145537 |
| SAP Data Services 4.2 | | | Nota de SAP 2288344 |
| Plataforma de comercio de Hybris SAP 5.x y 6.x |  Windows | SQL Server, Oracle | Todos los tipos de máquina virtual con certificado NetWeaver<br /> [Wiki de Hybris](https://wiki.hybris.com/display/SUP/Using+the+hybris+Platform+with+the+Cloud) |
