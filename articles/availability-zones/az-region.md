---
title: Regiones que admiten Availability Zones en Azure
description: Para crear aplicaciones altamente disponibles y resistentes en Azure, las zonas de disponibilidad proporcionan ubicaciones físicamente separadas que puede utilizar para ejecutar sus recursos.
author: cynthn
ms.service: azure
ms.topic: article
ms.date: 08/04/2020
ms.author: cynthn
ms.custom: fasttrack-edit, mvc, references_regions
ms.openlocfilehash: e13d7a1cb35d2b0c9544a8be36f834efc6970501
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87759655"
---
# <a name="regions-that-support-availability-zones-in-azure"></a>Regiones que admiten Availability Zones en Azure

Las zonas de disponibilidad son una oferta que protege las aplicaciones y datos de los errores del centro de datos. Para más información sobre Availability Zones, consulte [¿Qué son las zonas de disponibilidad en Azure?](az-overview.md)

## <a name="services-support-by-region"></a>Soporte técnico de servicios por región

En esta sección se enumeran los servicios y las regiones de Azure que admiten Availability Zones.

Los servicios que están disponibles en cada región, junto con la próxima hoja de ruta de disponibilidad, se pueden encontrar en [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/).

| Servicio |América: Centro de EE. UU.|América: Este de EE. UU.|América: Este de EE. UU. 2|América: Oeste de EE. UU. 2|Europa: Centro de Francia|Europa: Norte de Europa|Europa: Sur de Reino Unido|Europa: Oeste de Europa|Asia Pacífico: Japón Oriental|Asia Pacífico: Sudeste de Asia|Asia Pacífico: Este de Australia|
|----------------------------|----------|----------|---------|---------|--------------|------------|--------|----------|----------|-------------|-------------|
| **Proceso**                         |            |              |           |           |                |              |          |             |            |                |                |
| Máquinas virtuales Linux          | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       | &#10003;               |
| Máquinas virtuales Windows        | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       | &#10003;               |
| Virtual Machine Scale Sets      | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       | &#10003;               |
| Azure App Service Environment ILB | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       |                |
| Azure Kubernetes Service        | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       |                |
| **Storage**   |            |              |           |           |                |              |          |             |            |                |                |
| Managed Disks                   | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       | &#10003;               |
| Almacenamiento con redundancia de zona          | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       | &#10003;               |
| **Redes**                     |            |              |           |           |                |              |          |             |            |                |                |
| Dirección IP estándar        | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       | &#10003;               |
| Standard Load Balancer     | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       | &#10003;               |
| VPN Gateway            | &#10003;   |  &#10003;    | &#10003;  | &#10003;  | &#10003;       | &#10003;     |  &#10003;  | &#10003;    |  &#10003;   | &#10003;       | &#10003;               |
| Puerta de enlace de ExpressRoute   | &#10003;   |  &#10003;    | &#10003;  | &#10003;  | &#10003;       | &#10003;     |  &#10003;  | &#10003;    |  &#10003;   | &#10003;       | &#10003;               |
| Application Gateway (v2)    | &#10003;   |  &#10003;    | &#10003;  | &#10003;  | &#10003;       | &#10003;     |  &#10003;  | &#10003;    |  &#10003;   | &#10003;       | &#10003;               |
| Azure Firewall           | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    |  &#10003;       | &#10003;       |                |
| **Bases de datos**                     |            |              |           |           |                |              |          |             |            |                |                |
| Explorador de datos de Azure                   | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;        | &#10003;       |                |
| SQL Database                    | &#10003;   | &#10003;     | &#10003;  | &#10003;(versión preliminar) | &#10003;       | &#10003;(versión preliminar)     | &#10003; | &#10003;    | &#10003;       | &#10003;       |&#10003;        |
| Azure Cache for Redis           | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    |  &#10003;       | &#10003;       |                |
| Azure Cosmos DB      | &#10003;   |  &#10003;  |  &#10003; | &#10003; |  &#10003;  | &#10003;    | &#10003; |  &#10003;   |     &#10003;       | &#10003;    | &#10003;  |
| **Analytics**                       |            |              |           |           |                |              |          |             |            |                |                |
| Event Hubs                      | &#10003;   |   &#10003; | &#10003;  | &#10003;  | &#10003; | &#10003; | &#10003; | &#10003; | &#10003; | &#10003;       |  &#10003;              |
| **Integración**                     |            |              |           |           |                |              |          |             |            |                |                |
| Service Bus (solo nivel Premium) | &#10003;   |  &#10003;  | &#10003;  | &#10003;  | &#10003;  | &#10003;     |&#10003;   | &#10003;    |&#10003;      | &#10003;       |  &#10003;              |
| Event Grid | &#10003;   |  &#10003;  | &#10003;  | &#10003;  | &#10003;  | &#10003;     |&#10003;   | &#10003;    |&#10003;      | &#10003;       |                |
| **Identidad**                     |            |              |           |           |                |              |          |             |            |                |                |
| Azure AD Domain Services | &#10003;   |  &#10003;  | &#10003;  | &#10003;  | &#10003;  | &#10003;     |&#10003;   | &#10003;    |&#10003;      | &#10003;       |                |

Azure también ofrece compatibilidad con Availability Zones en las siguientes regiones:

- US Gov - Virginia
- Norte de Sudáfrica
- Centro-sur de EE. UU.
- Centro de Canadá

Para obtener más información sobre la compatibilidad de Availability Zones en estas cuatro regiones, póngase en contacto con el representante de ventas o de clientes de Microsoft o abra una solicitud de soporte técnico.

## <a name="next-steps"></a>Pasos siguientes

- [Regiones y Availability Zones en Azure](az-overview.md)
