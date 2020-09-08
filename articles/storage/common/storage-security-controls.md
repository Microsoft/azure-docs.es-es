---
title: Controles de seguridad
titleSuffix: Azure Storage
description: Vea las listas de comprobación de controles de seguridad para evaluar Azure Storage. Las áreas que estas listas abarcan son la protección de datos, la red, la supervisión y el registro, la identidad y la configuración.
services: storage
author: msmbaldwin
ms.author: mbaldwin
ms.service: storage
ms.subservice: common
ms.topic: conceptual
ms.date: 03/11/2020
ms.openlocfilehash: b816e7bd5e00b21700bc994fc0860195d39f2915
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87826524"
---
# <a name="security-controls-for-azure-storage"></a>Controles de seguridad para los servicios de Azure Storage

En este artículo, se explican los controles de seguridad integrados en Azure Storage.

[!INCLUDE [Security controls Header](../../../includes/security-controls-header.md)]

## <a name="data-protection"></a>Protección de los datos

| Control de seguridad | Sí/No | Notas |
|---|---|--|
| Cifrado del lado servidor en reposo: Claves administradas por Microsoft | Sí |  |
| Cifrado del lado servidor en reposo: claves administradas por el cliente (BYOK) | Sí | Consulte [Storage Service Encryption mediante claves administradas por el cliente en Azure Key Vault](storage-service-encryption-customer-managed-keys.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).|
| Cifrado de nivel de columna (Azure Data Services)| N/D |  |
| Cifrado en tránsito (por ejemplo, cifrado de ExpressRoute, cifrado en la red virtual y cifrado de red virtual a red virtual)| Sí | Compatible con los mecanismos estándar de HTTPS/TLS.  Los usuarios también pueden cifrar datos antes de que se transmitan al servicio. |
| Llamadas a API cifradas| Sí |  |

## <a name="network"></a>Red

| Control de seguridad | Sí/No | Notas |
|---|---|--|
| Compatibilidad con punto de conexión de servicio| Sí |  |
| Compatibilidad con etiquetas de servicio| Sí | Para más información sobre las etiquetas de servicio compatibles con Azure Storage, consulte [Introducción a las etiquetas de servicio de Azure](../../virtual-network/service-tags-overview.md). |
| Compatibilidad con la inserción de redes virtuales| N/D |  |
| Compatibilidad con el aislamiento de red y los firewalls| Sí | |
| Compatibilidad con la tunelización forzada| N/D |  |

## <a name="monitoring--logging"></a>Supervisión y registro

| Control de seguridad | Sí/No | Notas|
|---|---|--|
| Compatibilidad con la supervisión de Azure (Log Analytics, Application Insights, etc.)| Sí | Métricas de Azure Monitor|
| Registro y auditoría del plano de administración y de control | Sí | Azure Activity Log |
| Registro y auditoría del plano de datos| Sí | Registros de recursos de Azure Monitor |

## <a name="identity"></a>Identidad

| Control de seguridad | Sí/No | Notas|
|---|---|--|
| Authentication| Sí | Azure Active Directory, clave compartida, el token de acceso compartido. |
| Authorization| Sí | Compatible con la autorización mediante RBAC, listas de control de acceso de POSIX y tokens de SAS |

## <a name="configuration-management"></a>Administración de configuración

| Control de seguridad | Sí/No | Notas|
|---|---|--|
| Compatibilidad con la administración de configuración (control de versiones de configuración, etc.)| Sí | Compatible con el control de versiones del proveedor de recursos mediante las API de Azure Resource Manager |

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre los [controles de seguridad integrados en los servicios de Azure](../../security/fundamentals/security-controls.md).