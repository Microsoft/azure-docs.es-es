---
title: Certificaciones de Microsoft Azure para SAP | Microsoft Docs
description: Lista actualizada de las configuraciones y certificaciones actuales de SAP en la plataforma Azure.
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 04/21/2020
ms.author: juergent
ms.custom: ''
ms.openlocfilehash: f78d5f839c5dd7816ae28eeed175503427abb43c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87035849"
---
# <a name="sap-certifications-and-configurations-running-on-microsoft-azure"></a>Configuraciones y certificaciones de SAP que se ejecutan en Microsoft Azure

SAP y Microsoft tienen una larga historia de trabajo conjunto en una asociación segura que tiene ventajas mutuas para sus clientes. Microsoft actualiza constantemente su plataforma y envía nuevos detalles de certificación a SAP, con el fin de asegurarse de que Microsoft Azure es la mejor plataforma para ejecutar las cargas de trabajo de SAP. Las tablas siguientes describen nuestras configuraciones admitidas de Azure y una lista de certificaciones SAP en crecimiento. Esta lista es una lista de información general que puede desviarse aquí y allá de las listas de SAP oficiales. En el artículo [Qué software de SAP es compatible para su implementación en Azure](./sap-supported-product-on-azure.md) se documenta cómo llegar a los datos detallados.

## <a name="sap-hana-certifications"></a>Certificaciones SAP HANA
Referencias:

- [SAP HANA certified IaaS platforms](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) (Plataformas IaaS con certificación SAP HANA) para soporte técnico para máquinas virtuales de Azure nativas e instancias grandes de HANA.

| Producto de SAP | SO admitido | Productos de Azure |
| --- | --- | --- |
| SAP HANA Developer Edition (incluido el software cliente de HANA compuesto por los controladores SQLODBC, ODBO solo para Windows, ODBC y JDBC, HANA Studio y la base de datos HANA) | Red Hat Enterprise Linux, SUSE Linux Enterprise | Familia de máquina virtual de la serie D |
| Business One on HANA | SUSE Linux Enterprise | DS14_v2, M32ts, M32ls, M64ls, M64s <br /> [Plataformas IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure%23SAP%20Business%20One) |
| SAP S/4 HANA | Red Hat Enterprise Linux, SUSE Linux Enterprise | Disponibilidad controlada para GS5. Compatibilidad total con M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, M416s_v2, M416ms_v2, <br /> SAP HANA en Azure (instancias grandes) [Plataformas IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) |
| Suite en Hana, OLTP | Red Hat Enterprise Linux, SUSE Linux Enterprise | M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, <br /> M416s_v2, M416ms_v2, SAP HANA en Azure (instancias grandes) [Plataformas IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) |
| Hana Enterprise para BW, OLAP | Red Hat Enterprise Linux, SUSE Linux Enterprise | GS5, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, <br /> M416s_v2, M416ms_v2, SAP HANA en Azure (instancias grandes) [Plataformas IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) |
| SAP BW/4 HANA | Red Hat Enterprise Linux, SUSE Linux Enterprise | GS5, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, <br /> M416s_v2, M416ms_v2, SAP HANA en Azure (instancias grandes) <br /> [Plataformas IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) |

Tenga en cuenta que SAP utiliza el término "agrupación en clústeres" en las [Plataformas IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) como sinónimo para "escalabilidad horizontal" y NO para "agrupación en clústeres" de alta disponibilidad

## <a name="sap-netweaver-certifications"></a>Certificaciones de SAP NetWeaver
Microsoft Azure cuenta con certificación para los siguientes productos de SAP, con soporte técnico completo de Microsoft y SAP.
Referencias:

- [1928533 - Aplicaciones de SAP en Azure: tipos de máquina virtual de Azure y productos compatibles](https://launchpad.support.sap.com/#/notes/1928533) para todas las aplicaciones basadas en SAP NetWeaver, incluidas SAP TREX, SAP LiveCache y SAP Content Server. Y todas las bases de datos, excluida SAP HANA.


| Producto de SAP | SO invitado | RDBMS | Tipos de máquinas virtuales |
| --- | --- | --- | --- |
| Software SAP Business Suite | Windows, SUSE Linux Enterprise, Red Hat Enterprise Linux, Oracle Linux |SQL Server, Oracle (solo Windows y Oracle Linux), DB2, SAP ASE |A5 to A11, D11 to D14, DS11 to DS14, DS11_v2 to DS15_v2, GS1 to GS5, D2s_v3 to D64s_v3, D2as_v4 to D64as_v4, E2s_v3 to E64s_v3, E2as_v4 to E64as_v4, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, M416s_v2, M416ms_v2 |
| SAP Business All-in-One | Windows, SUSE Linux Enterprise, Red Hat Enterprise Linux, Oracle Linux |SQL Server, Oracle (solo Windows y Oracle Linux), DB2, SAP ASE |A5 to A11, D11 to D14, DS11 to DS14, DS11_v2 to DS15_v2, GS1 to GS5, D2s_v3 to D64s_v3, D2as_v4 to D64as_v4, E2s_v3 to E64s_v3, E2as_v4 to E64as_v4, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, M416s_v2, M416ms_v2 |
| SAP BusinessObjects BI | Windows |N/D |A5 to A11, D11 to D14, DS11 to DS14, DS11_v2 to DS15_v2, GS1 to GS5, D2s_v3 to D64s_v3, D2as_v4 to D64as_v4, E2s_v3 to E64s_v3, E2as_v4 to E64as_v4, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, M416s_v2, M416ms_v2 |
| SAP NetWeaver | Windows, SUSE Linux Enterprise, Red Hat Enterprise Linux, Oracle Linux |SQL Server, Oracle (solo Windows y Oracle Linux), DB2, SAP ASE |A5 to A11, D11 to D14, DS11 to DS14, DS11_v2 to DS15_v2, GS1 to GS5, D2s_v3 to D64s_v3, D2as_v4 to D64as_v4, E2s_v3 to E64s_v3, E2as_v4 to E64as_v4, M64s, M64ms, M128s, M128ms, M64ls, M32ls, M32ts, M208s_v2, M208ms_v2, M416s_v2, M416ms_v2 |

## <a name="other-sap-workload-supported-on-azure"></a>Otras cargas de trabajo de SAP admitidas en Azure

| Producto de SAP | SO invitado | RDBMS | Tipos de máquinas virtuales |
| --- | --- | --- | --- |
| SAP Business One on SQL Server | Windows  | SQL Server | Todos los tipos de máquina virtual con certificado NetWeaver<br /> [Nota de SAP #928839](https://launchpad.support.sap.com/#/notes/928839) |
| SAP BPC 10.01 MS SP08 | Windows y Linux | | Todos los tipos de máquina virtual con certificado NetWeaver<br /> Nota de SAP 2451795 |
| SAP Business Objects BI platform | Windows y Linux | | Nota de SAP 2145537 |
| SAP Data Services 4.2 | | | Nota de SAP 2288344 |
| Plataforma de comercio SAP Hybris  | Windows | SQL Server, Oracle | Todos los tipos de máquina virtual con certificado NetWeaver <br /> [Documentación de Hybris](https://help.sap.com/viewer/a74589c3a81a4a95bf51d87258c0ab15/6.7.0.0/en-US/8c71300f866910149b40c88dfc0de431.html) |
| Plataforma de comercio SAP Hybris  | SLES 12 o más reciente | SAP HANA | Todos los tipos de máquina virtual con certificado NetWeaver <br /> [Documentación de Hybris](https://help.sap.com/viewer/a74589c3a81a4a95bf51d87258c0ab15/6.7.0.0/en-US/8c71300f866910149b40c88dfc0de431.html) |
| Plataforma de comercio SAP Hybris  | RHEL 7 o posterior | SAP HANA | Todos los tipos de máquina virtual con certificado NetWeaver <br /> [Documentación de Hybris]https://help.sap.com/viewer/a74589c3a81a4a95bf51d87258c0ab15/6.7.0.0/en-US/8c71300f866910149b40c88dfc0de431.html) |
| Plataforma de comercio SAP Hybris 1811 y posterior  | Windows, SLES o RHEL | Base de datos de SQL Azure | Todos los tipos de máquina virtual con certificado NetWeaver <br /> [Documentación de Hybris](https://help.sap.com/viewer/a74589c3a81a4a95bf51d87258c0ab15/1811/en-US/8c71300f866910149b40c88dfc0de431.html) |
