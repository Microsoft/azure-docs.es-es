---
author: mmacy
ms.service: active-directory
ms.subservice: develop
ms.topic: include
ms.date: 07/27/2020
ms.author: marsma
ms.openlocfilehash: 99ca8e45da27f5bce5258d55f46bcfa25b358239
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87311555"
---
> [!IMPORTANT]
> MSAL.js 2.0 no admite actualmente el uso de Azure AD B2C con el flujo de código de autorización de PKCE. En este momento, Azure AD B2C recomienda usar el flujo implícito, tal y como se describe en [Tutorial: Registro de una aplicación][implicit-flow]. Para seguir el progreso de este problema, consulte la [Wiki de MSAL.js][msal-wiki].

[github-issue]: https://github.com/AzureAD/microsoft-authentication-library-for-js/issues/1795
[implicit-flow]: ../articles/active-directory-b2c/tutorial-register-applications.md
[msal-wiki]: https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/MSAL-browser-B2C-CORS-issue
