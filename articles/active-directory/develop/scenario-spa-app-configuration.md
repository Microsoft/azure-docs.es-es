---
title: 'Configuración de una aplicación de página única: Plataforma de identidad de Microsoft | Azure'
description: Aprenda a crear una aplicación de página única (configuración del código de la aplicación)
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 02/11/2020
ms.author: nacanuma
ms.custom: aaddev
ms.openlocfilehash: b42d720a425b92ec9002f7c2b9797a91f70dafe2
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89003034"
---
# <a name="single-page-application-code-configuration"></a>Aplicación de página única: Configuración del código

Obtenga información sobre cómo configurar el código de la aplicación de página única (SPA).

## <a name="msal-libraries-for-spas-and-supported-authentication-flows"></a>Bibliotecas de MSAL para aplicaciones de página única y flujos de autenticación compatibles

La plataforma de identidad de Microsoft proporciona la siguiente biblioteca de autenticación de Microsoft para JavaScript (MSAL.js) para admitir el flujo implícito y el flujo de código de autorización con PKCE mediante prácticas de seguridad recomendadas por el sector:

| Biblioteca MSAL | Flujo | Descripción |
|--------------|------|-------------|
| ![MSAL.js](media/sample-v2-code/logo_js.png) <br/> [MSAL.js (2.x)](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser) | Flujo de código de autorización (PKCE) | Biblioteca de JavaScript sin formato para usar en cualquier aplicación web cliente compilada con marcos de JavaScript o SPA, como Angular, Vue.js y React.js. |
| ![MSAL.js](media/sample-v2-code/logo_js.png) <br/> [MSAL.js (1.x)](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-core) | Flujo implícito | Biblioteca de JavaScript sin formato para usar en cualquier aplicación web cliente compilada con marcos de JavaScript o SPA, como Angular, Vue.js y React.js. |
| ![MSAL Angular](media/sample-v2-code/logo_angular.png) <br/> [MSAL Angular](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/README.md) | Flujo implícito | Contenedor de la biblioteca básica de MSAL.js para simplificar el uso en aplicaciones de página única creadas con el marco Angular. |

## <a name="application-code-configuration"></a>Configuración del código de la aplicación

En una biblioteca MSAL, la información de registro de aplicación se pasa como una configuración durante la inicialización de la biblioteca.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
// Configuration object constructed.
const config = {
    auth: {
        clientId: 'your_client_id'
    }
};

// create UserAgentApplication instance
const userAgentApplication = new UserAgentApplication(config);
```

Para más información sobre las opciones configurables, consulte [Inicialización de aplicaciones con MSAL.js](msal-js-initializing-client-applications.md).

# <a name="angular"></a>[Angular](#tab/angular)

```javascript
// App.module.ts
import { MsalModule } from '@azure/msal-angular';

@NgModule({
    imports: [
        MsalModule.forRoot({
            auth: {
                clientId: 'your_app_id'
            }
        })
    ]
})

export class AppModule { }
```

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Inicio y cierre de sesión](scenario-spa-sign-in.md)
