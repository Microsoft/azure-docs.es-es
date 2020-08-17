---
title: 'Controles de seguridad para Azure Linux Virtual Machines: Linux'
description: Lista de comprobación de los controles de seguridad que se usan para evaluar Azure Linux Virtual Machines
services: virtual-machines
ms.service: virtual-machines
documentationcenter: ''
author: msmbaldwin
manager: rkarlin
ms.topic: conceptual
ms.date: 09/04/2019
ms.author: mbaldwin
ms.openlocfilehash: 132f696dd1298384c302eeadc264e857d1edff96
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080070"
---
# <a name="security-controls-for-linux-virtual-machines"></a>Controles de seguridad para Linux Virtual Machines

En este artículo se documentan los controles de seguridad integrados en Linux Virtual Machines.

[!INCLUDE [Security controls header](../../../includes/security-controls-header.md)]

## <a name="network"></a>Red

| Control de seguridad | Sí/No | Notas |
|---|---|--|
| Compatibilidad con punto de conexión de servicio| Sí | |
| Compatibilidad con la inserción de redes virtuales| Sí | |
| Compatibilidad con el aislamiento de red y los firewalls| Sí |  |
| Compatibilidad con la tunelización forzada| Sí | Consulte [Configuración de la tunelización forzada mediante el modelo de implementación de Azure Resource Manager](../../vpn-gateway/vpn-gateway-forced-tunneling-rm.md). |

## <a name="monitoring--logging"></a>Supervisión y registro

| Control de seguridad | Sí/No | Notas|
|---|---|--|
| Compatibilidad con la supervisión de Azure (Log Analytics, Application Insights, etc.)| Sí | Consulte [Supervisión y actualización de una máquina virtual Linux en Azure](./tutorial-monitor.md). |
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
| Cifrado del lado servidor en reposo: Claves administradas por Microsoft | Sí | Consulte [Azure Disk Encryption para máquinas virtuales Linux](disk-encryption-overview.md). |
| Cifrado en tránsito (por ejemplo, cifrado de ExpressRoute, cifrado en la red virtual y cifrado de red virtual a red virtual)| Sí | Azure Virtual Machines admite [ExpressRoute](../../expressroute/index.yml) y el cifrado de red virtual. Consulte [Cifrado en tránsito en máquinas virtuales](../../security/fundamentals/encryption-overview.md#in-transit-encryption-in-vms). |
| Cifrado del lado servidor en reposo: claves administradas por el cliente (BYOK) | Sí | Las claves administradas por el cliente es un escenario de cifrado admitidos de Azure; consulte [Introducción al cifrado de Azure](../../security/fundamentals/encryption-overview.md#in-transit-encryption-in-vms).|
| Cifrado de nivel de columna (Azure Data Services)| N/D | |
| Llamadas a API cifradas| Sí | A través de HTTPS y TLS. |

## <a name="configuration-management"></a>Administración de configuración

| Control de seguridad | Sí/No | Notas|
|---|---|--|
| Compatibilidad con la administración de configuración (control de versiones de configuración, etc.)| Sí |  | 

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre los [controles de seguridad integrados en los servicios de Azure](../../security/fundamentals/security-controls.md).
