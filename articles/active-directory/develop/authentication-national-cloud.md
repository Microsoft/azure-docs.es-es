---
title: Autenticación en Azure AD y nubes nacionales | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información acerca de los puntos de conexión para la autenticación y registro de aplicaciones en nubes nacionales.
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/28/2019
ms.author: negoe
ms.reviewer: negoe,celested
ms.custom: aaddev,references_regions
ms.openlocfilehash: d27498748f03fb101d165351ec80c77d57f78f1a
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88141506"
---
# <a name="national-clouds"></a>Nubes nacionales

Las nubes nacionales son instancias físicamente aisladas de Azure. Estas regiones de Azure se han diseñado para asegurarse de que cumplen los requisitos de residencia, soberanía y cumplimiento normativo de los datos dentro de las fronteras geográficas.

Incluida la nube mundial, Azure Active Directory (Azure AD) se implementa en las siguientes nubes nacionales:  

- Azure Government
- Azure Alemania
- Azure China 21Vianet

Las nubes nacionales son únicas y están en un entorno independiente de la nube mundial de Azure. Es importante tener en cuenta las diferencias clave al desarrollar la aplicación para estos entornos. Entre las diferencias se incluyen el registro de aplicaciones, la adquisición de tokens y la configuración de puntos de conexión.

## <a name="app-registration-endpoints"></a>Puntos de conexión de registro de aplicaciones

Hay una instancia de Azure Portal independiente para cada una de las nubes nacionales. Para integrar aplicaciones con la plataforma de identidad de Microsoft en una nube nacional, deberá registrar la aplicación por separado en cada instancia de Azure Portal específica del entorno.

En la tabla siguiente se enumeran las direcciones URL base de los puntos de conexión de Azure AD que se usan para registrar una aplicación en cada nube nacional.

| Nube nacional | Punto de conexión de portal de Azure AD |
|----------------|--------------------------|
| Azure AD para el US Gov | `https://portal.azure.us` |
| Azure AD Alemania | `https://portal.microsoftazure.de` |
| Azure AD China, operado por 21Vianet | `https://portal.azure.cn` |
| Azure AD (servicio global) |`https://portal.azure.com` |

## <a name="azure-ad-authentication-endpoints"></a>Puntos de conexión de autenticación de Azure AD

Todas las nubes nacionales autentican a los usuarios por separado en cada entorno y tienen puntos de conexión de autenticación independientes.

En la tabla siguiente se enumeran las direcciones URL base de los puntos de conexión de Azure AD que se usan para adquirir tokens en cada nube nacional.

| Nube nacional | Punto de conexión de autenticación de Azure AD |
|----------------|-------------------------|
| Azure AD para el US Gov | `https://login.microsoftonline.us` |
| Azure AD Alemania| `https://login.microsoftonline.de` |
| Azure AD China, operado por 21Vianet | `https://login.chinacloudapi.cn` |
| Azure AD (servicio global)| `https://login.microsoftonline.com` |

Puede realizar solicitudes a los puntos de conexión de autorización o token de Azure AD mediante la dirección URL base específica de la región pertinente. Por ejemplo, para Azure Alemania:

  - El punto de conexión común de autorización es `https://login.microsoftonline.de/common/oauth2/v2.0/authorize`.
  - El punto de conexión común de token es `https://login.microsoftonline.de/common/oauth2/v2.0/token`.

En el caso de las aplicaciones de inquilino único, reemplace "common" en las direcciones URL anteriores por el identificador o el nombre del inquilino. Un ejemplo es `https://login.microsoftonline.de/contoso.com`.

## <a name="microsoft-graph-api"></a>Microsoft Graph API

Para obtener información sobre cómo llamar a las API de Microsoft Graph en el entorno de nube nacional, vaya a [Microsoft Graph en implementaciones de nube nacional](/graph/deployments).

> [!IMPORTANT]
> Ciertos servicios y características que se encuentran en regiones específicas del servicio mundial pueden no estar disponibles en todas las nubes nacionales. Para saber qué servicios están disponibles, consulte [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=all&regions=usgov-non-regional,us-dod-central,us-dod-east,usgov-arizona,usgov-iowa,usgov-texas,usgov-virginia,china-non-regional,china-east,china-east-2,china-north,china-north-2,germany-non-regional,germany-central,germany-northeast).

Para aprender a compilar una aplicación mediante la plataforma de identidad de Microsoft, siga el [Tutorial de la biblioteca de autenticación de Microsoft (MSAL)](msal-national-cloud.md). En concreto, esta aplicación iniciará la sesión de un usuario y obtendrá un token de acceso para llamar a la API de Microsoft Graph.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre:

- [Azure Government](../../azure-government/index.yml)
- [Azure China 21Vianet](/azure/china/)
- [Azure Alemania](../../germany/index.yml)
- [Conceptos básicos sobre la autenticación de Azure AD](./authentication-vs-authorization.md)