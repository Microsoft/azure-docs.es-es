---
title: Novedades de Azure AD en Microsoft 365 Government | Microsoft Docs
description: Obtenga información sobre algunos cambios de Azure Active Directory (Azure AD) en la instancia en la nube de Microsoft 365 Government, que podrían afectarle.
services: active-directory
author: eross-msft
manager: daveba
ms.author: lizross
ms.reviewer: sumitp
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 05/07/2019
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: a0efc4bc8f89b0fbefbba171d80a3f8a1ed5e7f6
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89318937"
---
# <a name="whats-new-for-azure-active-directory-in-microsoft-365-government"></a>Novedades de Azure Active Directory en Microsoft 365 Government

Hemos realizado algunos cambios en Azure Active Directory (Azure AD) en la instancia en la nube de Microsoft 365 Government, que es aplicable a los clientes que usan los siguientes servicios:

- Microsoft Azure Government

- Microsoft 365 Government – GCC High

- Microsoft 365 Government – DoD

Este artículo no se aplica a los clientes de Microsoft 365 Government – GCC.

## <a name="changes-to-the-initial-domain-name"></a>Cambios realizados en el nombre de dominio inicial

Durante la suscripción inicial de su organización a un servicio en línea de Microsoft 365 Government, se le pidió que eligiera el nombre de dominio de su organización, `<your-domain-name>.onmicrosoft.com`. Si ya tiene un nombre de dominio con el sufijo .com, nada cambiará.

Sin embargo, si está registrándose para un nuevo servicio de Microsoft 365 Government, le pedirá que elija un nombre de dominio con el sufijo `.us`. Por lo tanto, será `<your-domain-name>.onmicrosoft.us`.

>[!Note]
>Este cambio no se aplica a todos los clientes que están administrados por los proveedores de servicios en la nube (CSP).

## <a name="changes-to-portal-access"></a>Cambios realizados en el acceso al portal

Hemos actualizado los puntos de conexión del portal de Microsoft Azure Government, Microsoft 365 Government – GCC High y Microsoft 365 Government – DoD, como se muestra en la [tabla de asignación de puntos de conexión](#endpoint-mapping).

Antes, los clientes podían iniciar sesión en cualquier parte del mundo con el portal de Azure (portal.azure.com) y con el portal de Office 365 (portal.office.com). Con esta actualización, ahora los clientes deben iniciar sesión con los portales específicos de Microsoft Azure Government, Microsoft 365 Government - GCC High y Microsoft 365 Government - DoD.

## <a name="endpoint-mapping"></a>Asignación de puntos de conexión

En la siguiente tabla se muestran los puntos de conexión para todos los clientes:

| Nombre | Detalles del punto de conexión |
|------|------------------|
| Portales |Microsoft Azure Government: https://portal.azure.us<p>Microsoft 365 Government – GCC High: https://portal.office365.us<p>Microsoft 365 Government – DoD: https://portal.apps.mil |
| Punto de conexión de Azure Active Directory Authority | https://login.microsoftonline.us |
| Microsoft Graph API para Microsoft 365 Government - GCC High | https://graph.microsoft.us |
| Microsoft Graph API para Microsoft 365 Government - DoD | https://dod-graph.microsoft.us |
| Puntos de conexión de servicio de Azure Government | Para obtener más información, vea [Guía para desarrolladores de Azure Government](../../azure-government/documentation-government-developer-guide.md) |
| Puntos de conexión de Microsoft 365 Government - GCC High | Para obtener más información, consulte [Office 365 U.S. Government GCC High endpoints](/office365/enterprise/office-365-u-s-government-gcc-high-endpoints) (Puntos de conexión de Office 365 U.S. Government GCC High) |
| Microsoft 365 Government - DoD | Para obtener más información, consulte [Office 365 U.S. Government DoD endpoints](/office365/enterprise/office-365-u-s-government-dod-endpoints) (Puntos de conexión de Office 365 U.S. Government DoD) |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte estos artículos:

- [¿Qué es Azure Government?](../../azure-government/documentation-government-welcome.md)

- [Azure Government AAD Authority Endpoint Update](https://devblogs.microsoft.com/azuregov/azure-government-aad-authority-endpoint-update/) (Actualización del punto de conexión de la entidad de AAD de Azure Government)

- [Puntos de conexión de Microsoft Graph en la nube del US Gov](https://developer.microsoft.com/graph/blogs/new-microsoft-graph-endpoints-in-us-government-cloud/)

- [Office 365 US Government GCC High y DoD](/office365/servicedescriptions/office-365-platform-service-description/office-365-us-government/gcc-high-and-dod)