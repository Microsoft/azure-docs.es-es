---
title: Comportamiento de solicitudes interactivas (biblioteca de autenticación de Microsoft para JavaScript) | Azure
description: Obtenga información acerca de cómo personalizar el comportamiento de petición en las llamadas interactivas mediante la biblioteca de autenticación de Microsoft para JavaScript (MSAL.js).
services: active-directory
documentationcenter: dev-center-name
author: navyasric
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/24/2019
ms.author: nacanuma
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: dd0d736345f312f1a1d6f8f029b41429a3e5f0a7
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544267"
---
# <a name="prompt-behavior-in-msaljs-interactive-requests"></a>Comportamiento de solicitud en solicitudes interactivas MSAL.js

Cuando un usuario ha establecido la sesión de AD con varias cuentas de usuario de Azure active, la página de inicio de sesión de Azure AD le pedirá al usuario que seleccione una cuenta antes de iniciar sesión de forma predeterminada. Los usuarios no verán una selección de cuentas de experimentar si hay una sola sesión autenticada con Azure AD.

La biblioteca MSAL.js (a partir de v0.2.4) no envía un parámetro de símbolo del sistema durante las solicitudes interactivas (`loginRedirect`, `loginPopup`, `acquireTokenRedirect` y `acquireTokenPopup`) y, por tanto, no aplica ningún comportamiento de petición. Para las solicitudes de token silenciosas con el `acquireTokenSilent` MSAL.js de método, pasa un parámetro prompt establecido en `none`.

En función del escenario de aplicación, puede controlar el comportamiento del mensaje para las solicitudes interactivas estableciendo el parámetro de confirmación en los parámetros de solicitud se pasa a los métodos. Por ejemplo, si desea invocar la experiencia de selección de cuenta:

```javascript
var request = {
    scopes: ["user.read"],
    prompt: 'select_account',
}

userAgentApplication.loginRedirect(request);
```


Los siguientes valores de símbolo del sistema pueden pasarse al autenticarse con Azure AD:

**login:** Este valor obligará al usuario que escriba las credenciales en la solicitud de autenticación.

**select_account:** Este valor proporcionará al usuario una experiencia de selección de cuenta mostrar todas las cuentas en la sesión.

**consent:** Este valor invocará en el cuadro de diálogo de consentimiento de OAuth que permite a los usuarios conceder permisos a la aplicación.

**Ninguno:** Este valor se asegurará de que el usuario no ve ninguna solicitud interactiva. Se recomienda no pasar este valor a los métodos interactivos en MSAL.js porque puede tiene comportamientos inesperados. En su lugar, use el `acquireTokenSilent` método para lograr llamadas silenciosas.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la `prompt` parámetro en el [concesión implícita de OAuth 2.0](v2-oauth2-implicit-grant-flow.md) protocolo que usa la biblioteca MSAL.js.
