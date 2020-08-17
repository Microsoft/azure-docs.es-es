---
title: Introducción a los grupos de disponibilidad Always On de SQL Server
description: En este artículo se describen los grupos de disponibilidad AlwaysOn de SQL Server en Azure Virtual Machines.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.assetid: 601eebb1-fc2c-4f5b-9c05-0e6ffd0e5334
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 01/13/2017
ms.author: mathoma
ms.custom: seo-lt-2019, devx-track-azurecli
ms.openlocfilehash: bc0d5b0cdf616904ee7ed61fdc681ba60bff8f75
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87480295"
---
# <a name="introducing-sql-server-always-on-availability-groups-on-azure-virtual-machines"></a>Introducción a los grupos de disponibilidad Always On de SQL Server en Azure Virtual Machines

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se describen los grupos de disponibilidad de SQL Server en Azure Virtual Machines. 

Los grupos de disponibilidad AlwaysOn en Azure Virtual Machines son similares a los grupos de disponibilidad AlwaysOn locales. Para obtener más información, consulte [Grupos de disponibilidad AlwaysOn (SQL Server)](https://msdn.microsoft.com/library/hh510230.aspx). 

El diagrama muestra las partes de un grupo de disponibilidad de SQL Server completo en Azure Virtual Machines.

![Grupo de disponibilidad](./media/availability-group-overview/00-EndstateSampleNoELB.png)

La diferencia clave para un grupo de disponibilidad en Azure Virtual Machines es que las máquinas virtuales requieren un [equilibrador de carga](../../../load-balancer/load-balancer-overview.md). El equilibrador de carga almacena las direcciones IP del agente de escucha del grupo de disponibilidad. Si tiene más de un grupo de disponibilidad, necesita un cliente de escucha por cada grupo. Un equilibrador de carga puede admitir varios agentes de escucha.

Además, en un clúster de conmutación por error invitado de VM de IaaS de Azure, se recomienda una sola NIC por servidor (nodo de clúster) y una sola subred. La red de Azure tiene redundancia física, que hace que las NIC y subredes adicionales sean innecesarias en un clúster invitado de VM de IaaS de Azure. Aunque el informe de validación del clúster emita una advertencia acerca de que los nodos solo son accesibles en una única red, esta advertencia puede omitirse en los clústeres de conmutación por error invitados de VM de IaaS de Azure. 

Para aumentar la redundancia y la alta disponibilidad, las máquinas virtuales de SQL Server deben estar en el mismo [conjunto de disponibilidad](availability-group-manually-configure-prerequisites-tutorial.md#create-availability-sets) o en distintas [zonas de disponibilidad](/azure/availability-zones/az-overview). 

|  | Versión de Windows Server | SQL Server Version | Edición de SQL Server | Configuración de quórum WSFC | Recuperación ante desastres con varias regiones | Compatibilidad con varias subredes | Compatibilidad con un dominio de aplicación existente | Recuperación ante desastres con la misma región multizona | Compatibilidad de Dist-AG sin ningún dominio de AD | Compatibilidad de Dist-AG sin ningún clúster |  
| :------ | :-----| :-----| :-----| :-----| :-----| :-----| :-----| :-----| :-----| :-----|
| **[CLI de la máquina virtual de SQL](availability-group-az-cli-configure.md)** | 2016 | 2017 </br>2016   | Ent | Testigo en la nube | No | Sí | Sí | Sí | No | No |
| **[Plantillas de inicio rápido](availability-group-quickstart-template-configure.md)** | 2016 | 2017</br>2016  | Ent | Testigo en la nube | No | Sí | Sí | Sí | No | No |
| **[Manual](availability-group-manually-configure-prerequisites-tutorial.md)** | All | All | All | All | Sí | Sí | Sí | Sí | Sí | Sí |

La plantilla **SQL Server AlwaysOn Cluster (preview)** se ha quitado de Azure Marketplace y ya no está disponible. 

Cuando esté listo para crear un grupo de disponibilidad de SQL Server en Azure Virtual Machines, consulte estos tutoriales.

## <a name="manually-with-azure-cli"></a>Manualmente con la CLI de Azure

Se recomienda usar la CLI de Azure para configurar e implementar un grupo de disponibilidad porque tiene la implementación más rápida y sencilla. Con la CLI de Azure, tanto la creación del clúster de conmutación por error de Windows, que une máquinas virtuales de SQL Server al clúster, como la creación del agente de escucha y de Load Balancer interno pueden lograrse en menos de 30 minutos. Esta opción sigue requiriendo la creación manual del grupo de disponibilidad, pero automatiza todos los demás pasos de configuración necesarios. 

Para obtener más información, consulte [Usar la CLI de máquina virtual de Azure SQL para configurar un grupo de disponibilidad Always On para SQL Server en una máquina virtual de Azure](availability-group-az-cli-configure.md). 

## <a name="automatically-with-azure-quickstart-templates"></a>Automáticamente con plantillas de inicio rápido de Azure

Las plantillas de inicio rápido de Azure usan el proveedor de recursos de VM con SQL para implementar el clúster de conmutación por error de Windows, unir máquinas virtuales de SQL Server a este, crear el agente de escucha y configurar Load Balancer interno. Esta opción sigue necesitando que se creen manualmente el grupo de disponibilidad y el equilibrador de carga interno. Sin embargo, automatiza y simplifica todos los demás pasos de configuración necesarios, incluida la configuración del equilibrado de carga interno. 

Para obtener más información, consulte [Usar la plantilla de inicio rápido de Azure para configurar un grupo de disponibilidad Always On para SQL Server en una máquina virtual de Azure](availability-group-quickstart-template-configure.md).


## <a name="automatically-with-an-azure-portal-template"></a>Automáticamente con una plantilla de Azure Portal

[Configure Always On availability group in Azure VM automatically - Resource Manager](availability-group-azure-marketplace-template-configure.md) (Configuración automática de grupos de disponibilidad Always On de máquinas virtuales de Azure - Resource Manager)


## <a name="manually-in-the-azure-portal"></a>Manualmente en Azure Portal

También puede crear las máquinas virtuales sin la plantilla. En primer lugar, complete los requisitos previos y después cree el grupo de disponibilidad. Vea los siguientes temas: 

- [Configure prerequisites for SQL Server Always On availability groups on Azure Virtual Machines](availability-group-manually-configure-prerequisites-tutorial.md) (Configuración de requisitos previos para grupos de disponibilidad de SQL Server AlwaysOn en Azure Virtual Machines)///

- [Creación de un grupo de disponibilidad AlwaysOn para mejorar la disponibilidad y recuperación ante desastres](availability-group-manually-configure-tutorial.md)

## <a name="next-steps"></a>Pasos siguientes

[Configuración de un grupo de disponibilidad de SQL Server AlwaysOn en Azure Virtual Machines en regiones distintas](availability-group-manually-configure-multiple-regions.md)
