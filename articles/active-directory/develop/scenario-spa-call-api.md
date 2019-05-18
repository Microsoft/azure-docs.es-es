---
title: 'Aplicación de página única (llamada a una API web): la plataforma de identidad de Microsoft'
description: Aprenda a crear una aplicación de página única (llamada a una API web)
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/06/2019
ms.author: ryanwi
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 77a4ed01ac55a1153a62c672b33056a543b912ed
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545608"
---
# <a name="single-page-application---call-a-web-api"></a>Aplicación de página única - llamadas a API web

Se recomienda que llame a la `acquireTokenSilent` método para adquirir o renovar un token de acceso antes de llamar a una API web. Una vez que un token, puede llamar a una API web protegida.

## <a name="call-a-web-api"></a>Llamada a una API de web

### <a name="javascript"></a>JavaScript

Utilice el token de acceso adquirido como un portador en una solicitud HTTP para llamar a cualquier API web, como Microsoft Graph API. Por ejemplo:

```javascript
    var headers = new Headers();
    var bearer = "Bearer " + access_token;
    headers.append("Authorization", bearer);
    var options = {
         method: "GET",
         headers: headers
    };
    var graphEndpoint = "https://graph.microsoft.com/v1.0/me";

    fetch(graphEndpoint, options)
        .then(function (response) {
             //do something with response
        }
```

### <a name="angular"></a>Angular

Como se mencionó en el [al adquirir la sección tokens](scenario-spa-acquire-token.md), el contenedor MSAL Angular aprovecha el interceptor de HTTP para adquirir tokens de acceso de forma silenciosa y adjuntarlos a las solicitudes HTTP a las API automáticamente.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Pasar a producción](scenario-spa-production.md)
