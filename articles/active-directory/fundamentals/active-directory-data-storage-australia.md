---
title: 'Almacenamiento de datos de identidad para clientes australianos y neozelandeses: Azure AD'
description: Obtenga información sobre dónde almacena Azure Active Directory los datos relacionados con la identidad de los clientes australianos.
services: active-directory
author: ajburnle
manager: daveba
ms.author: ajburnle
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 12/13/2019
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 529c0ffa6b629dee124236e334c862c049f472ad
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89318546"
---
# <a name="identity-data-storage-for-australian-and-new-zealand-customers-in-azure-active-directory"></a>Almacenamiento de datos de identidad para clientes australianos y neozelandeses en Azure Active Directory

Azure AD almacena los datos de identidad en una ubicación geográfica en función de la dirección que proporcione la organización al suscribirse a un servicio de Microsoft Online, como Office 365 y Azure. Para más información sobre dónde se almacenan los datos de identidad del cliente, puede consultar la sección [Dónde se encuentran tus datos](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) del Centro de confianza de Microsoft.

> [!NOTE]
> Los servicios y las aplicaciones que se integran en Azure AD tienen acceso a datos de identidad del cliente. Evalúe cada servicio y aplicación que use para determinar la manera en que cada servicio y aplicación específico procesa los datos de identidad del cliente, y si cumple los requisitos de almacenamiento de datos de la empresa. Para obtener más información sobre la residencia de los datos de los servicios de Microsoft, consulte la sección Dónde se encuentran sus datos del Centro de confianza de Microsoft.

En el caso de los clientes que han proporcionado una dirección de Australia o Nueva Zelanda, Azure AD mantiene los datos de identidad de estos servicios en centros de datos australianos: 
- Administración de directorios de Azure AD 
- Authentication

Todos los demás servicios de Azure AD almacenan los datos del cliente en centros de datos globales. Para localizar el centro de datos de un servicio, consulte [Azure Active Directory: Dónde se encuentran sus datos](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located).

## <a name="microsoft-azure-multi-factor-authentication-mfa"></a>Microsoft Azure Multi-Factor Authentication (MFA)

MFA almacena datos de identidad del cliente en centros de datos globales. Para más información sobre los datos del usuario que Azure MFA basado en la nube recopila y almacena, consulte [Recopilación de datos de usuario de Azure Multi-Factor Authentication](../authentication/concept-mfa-data-residency.md).

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre cualquiera de las características y funcionalidades que se han descrito anteriormente, consulte estos artículos:
- [¿Qué es Multi-Factor Authentication?](../authentication/concept-mfa-howitworks.md)