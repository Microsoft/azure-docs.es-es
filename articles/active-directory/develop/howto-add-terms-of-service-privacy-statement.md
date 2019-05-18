---
title: Condiciones de servicio y declaración de privacidad de aplicaciones | Azure
description: Aprenda a configurar las condiciones del servicio y la declaración de privacidad de las aplicaciones registradas para usar Azure AD.
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: ryanwi
ms.reviwer: lenalepa, sureshja
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: b2242c6fc46f5556de0b0dd63659670e9c3f998d
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540158"
---
# <a name="how-to-configure-terms-of-service-and-privacy-statement-for-an-app"></a>Procedimientos para: Configurar las condiciones de servicio y declaración de privacidad de una aplicación

Los desarrolladores que crean y administran las aplicaciones que se integran con Azure Active Directory (Azure AD) y cuentas Microsoft deben incluir vínculos a las declaraciones del servicio y la declaración de privacidad de la aplicación. Las condiciones del servicio y la declaración de privacidad se exponen a los usuarios mediante la experiencia de consentimiento del usuario. Ayudan a los usuarios a saber que pueden confiar en la aplicación. Las condiciones del servicio y la declaración de privacidad son especialmente importantes para las aplicaciones multiinquilino orientadas al usuario: aplicaciones utilizadas por varios directorios o que están disponibles para cualquier cuenta Microsoft.

Es responsable de crear los documentos de las condiciones del servicio y de la declaración de privacidad para la aplicación, así como de proporcionar URL para estos documentos. Para las aplicaciones multiinquilino que no proporcionan estos vínculos, la experiencia de consentimiento del usuario para la aplicación mostrará una alerta que puede desaconsejar a los usuarios dar su consentimiento a la aplicación.

> [!NOTE]
> * Las aplicaciones de inquilino único no mostrarán ninguna alerta.
> * Si faltan uno o los dos vínculos, la aplicación mostrará una alerta.

## <a name="user-consent-experience"></a>Experiencia de consentimiento de usuario

Los ejemplos siguientes muestran la experiencia de consentimiento del usuario cuando se configuran las condiciones del servicio y la declaración de privacidad, y cuando estos vínculos no están configurados.

![Capturas de pantalla con y sin declaración de privacidad y condiciones del servicio proporcionados](./media/howto-add-terms-of-service-privacy-statement/user-consent-exp-privacy-statement-terms-service.png)

## <a name="formatting-links-to-the-terms-of-service-and-privacy-statement-documents"></a>Aplicación de formato a los vínculos de los documentos de las condiciones del servicio y de la declaración de privacidad

Antes de agregar vínculos a los documentos de las condiciones del servicio y de la declaración de privacidad para la aplicación, asegúrese de que las direcciones URL siguen estas directrices.

| Directrices     | DESCRIPCIÓN                           |
|---------------|---------------------------------------|
| Formato        | Dirección URL válida                             |
| Esquemas válidos | HTTP y HTTPS<br/>Se recomienda utilizar HTTPS |
| Longitud máxima    | 2048 caracteres                       |

Ejemplos: `https://myapp.com/terms-of-service` y `https://myapp.com/privacy-statement`

## <a name="adding-links-to-the-terms-of-service-and-privacy-statement"></a>Incorporación de vínculos a las condiciones del servicio y a la declaración de privacidad

Cuando las condiciones del servicio y la declaración de privacidad estén preparados, puede agregar vínculos a estos documentos en la aplicación mediante uno de estos métodos:

* [Mediante Azure Portal](#registered-in-azure-portal)
* [En el Portal de registro de aplicaciones o el Centro de desarrollo](#registered-in-app-reg-portal)
* [Con JSON del objeto de aplicación](#app-object-json)
* [Con la versión beta de la API REST MSGraph](#msgraph-beta-rest-api)

### <a name="registered-in-azure-portal"></a>Si ha registrado su aplicación en Azure Portal

Siga estos pasos si ha registrado su aplicación en Azure Portal.

1. Inicie sesión en el [Azure Portal](https://portal.azure.com/).
2. Navegue hasta la sección **Registros de aplicaciones** y seleccione la aplicación.
3. Abra la sección **Propiedades** de la aplicación.
4. Rellene los campos **URL de las condiciones del servicio** y **URL de la declaración de privacidad**.
5. Guarde los cambios.

    ![Sección de propiedades de aplicación con URL de las condiciones del servicio y de la declaración de privacidad](./media/howto-add-terms-of-service-privacy-statement/azure-portal-terms-service-privacy-statement-urls.png)

### <a name="registered-in-app-reg-portal"></a>Si ha registrado su aplicación en el Portal de registro de aplicaciones

Siga estos pasos si ha registrado su aplicación en el Portal de registro de aplicaciones o el Centro de desarrollo.

1. Inicie sesión en el [Portal de registro de aplicaciones](https://apps.dev.microsoft.com/).
2. Seleccione la aplicación y desplácese a la sección **Perfil**.
3. Rellene los campos **URL de las condiciones del servicio** y **URL de la declaración de privacidad**.
4. Guarde los cambios.

    ![Sección de perfil de aplicación con URL de las condiciones del servicio y de la declaración de privacidad](./media/howto-add-terms-of-service-privacy-statement/app-registration-portal-profile-terms-service-privacy-statement-urls.png)

### <a name="app-object-json"></a>Con JSON del objeto de aplicación

Si desea modificar directamente JSON del objeto de aplicación, puede usar el editor de manifiestos en Azure Portal o el Portal de registro de aplicaciones para incluir vínculos en las condiciones del servicio y la declaración de privacidad de la aplicación.

```json
    "informationalUrls": { 
        "termsOfService": "<your_terms_of_service_url>", 
        "privacy": "<your_privacy_statement_url>" 
    }
```

### <a name="msgraph-beta-rest-api"></a>Con API REST de la versión beta de MSGraph

Para actualizar mediante programación todas las aplicaciones, puede usar API REST de la versión beta de MSGraph para actualizar todas las aplicaciones y que incluyan vínculos en los documentos de las condiciones del servicio y de la declaración de privacidad.

```
PATCH https://graph.microsoft.com/beta/applications/{application id}
{ 
    "appId": "{your application id}", 
    "info": { 
        "termsOfServiceUrl": "<your_terms_of_service_url>", 
        "supportUrl": null, 
        "privacyStatementUrl": "<your_privacy_statement_url>", 
        "marketingUrl": null, 
        "logoUrl": null 
    }
}
```

> [!NOTE]
> * Tenga cuidado de no sobrescribir los valores previamente existentes que haya asignado a cualquiera de estos campos: `supportUrl`, `marketingUrl` y `logoUrl`
> * La API REST de la versión beta de MSGraph solo funcionará al iniciar sesión con una cuenta de Azure AD. No se admiten cuentas Microsoft personales.
