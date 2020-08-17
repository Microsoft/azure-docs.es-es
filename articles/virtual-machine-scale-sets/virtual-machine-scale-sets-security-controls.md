---
title: Controles de seguridad para Azure Virtual Machine Scale Sets
description: Lista de comprobación de los controles de seguridad para evaluar Azure Virtual Machine Scale Sets
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: security
ms.date: 09/05/2019
ms.reviewer: jushiman
ms.custom: mimckitt
ms.openlocfilehash: 4480ad425d9a3953fd5779f99d27b5b6b037e61e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87029423"
---
# <a name="security-controls-for-azure-virtual-machine-scale-sets"></a>Controles de seguridad para Azure Virtual Machine Scale Sets

En este artículo se explican los controles de seguridad integrados en Azure Virtual Machine Scale Sets.

[!INCLUDE [Security controls header](../../includes/security-controls-header.md)]

## <a name="network"></a>Red

| Control de seguridad | Sí/No | Notas |
|---|---|--|
| Compatibilidad con punto de conexión de servicio| Sí | |
| Compatibilidad con la inserción de redes virtuales| Sí | |
| Compatibilidad con el aislamiento de red y los firewalls| Sí |  |
| Compatibilidad con la tunelización forzada| Sí | Consulte [Configuración de la tunelización forzada mediante el modelo de implementación de Azure Resource Manager](../vpn-gateway/vpn-gateway-forced-tunneling-rm.md). |

## <a name="monitoring--logging"></a>Supervisión y registro

| Control de seguridad | Sí/No | Notas|
|---|---|--|
| Compatibilidad con la supervisión de Azure (Log Analytics, Application Insights, etc.)| Sí | Consulte [Supervisión y actualización de una máquina virtual Linux en Azure](../virtual-machines/linux/tutorial-monitor.md) y [Supervisión y actualización de una máquina virtual Windows en Azure](../virtual-machines/windows/tutorial-monitor.md). |
| Registro y auditoría del plano de administración y de control| Sí |  |
| Registro y auditoría del plano de datos | No |  |

## <a name="identity"></a>Identidad

| Control de seguridad | Sí/No | Notas|
|---|---|--|
| Authentication| Sí |  |
| Authorization| Sí |  |

## <a name="data-protection"></a>Protección de los datos

| Control de seguridad | Sí/No | Notas |
|---|---|--|
| Cifrado del lado servidor en reposo: Claves administradas por Microsoft | Sí | Consulte [Azure Disk Encryption para conjuntos de escalado de máquinas virtuales](disk-encryption-overview.md). |
| Cifrado en tránsito (por ejemplo, cifrado de ExpressRoute, cifrado en la red virtual y cifrado de red virtual a red virtual)| Sí | Azure Virtual Machines admite [ExpressRoute](../expressroute/index.yml) y el cifrado de red virtual. Consulte [Cifrado en tránsito en máquinas virtuales](../security/fundamentals/encryption-overview.md#in-transit-encryption-in-vms). |
| Cifrado del lado servidor en reposo: claves administradas por el cliente (BYOK) | Sí | Las claves administradas por el cliente es un escenario de cifrado admitidos de Azure; consulte [Azure Disk Encryption para conjuntos de escalado de máquinas virtuales](disk-encryption-overview.md).|
| Cifrado de nivel de columna (Azure Data Services)| N/D | |
| Llamadas a API cifradas| Sí | A través de HTTPS y TLS. |

## <a name="configuration-management"></a>Administración de configuración

| Control de seguridad | Sí/No | Notas|
|---|---|--|
| Compatibilidad con la administración de configuración (control de versiones de configuración, etc.)| Sí |  | 

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre los [controles de seguridad integrados en los servicios de Azure](../security/fundamentals/security-controls.md).
