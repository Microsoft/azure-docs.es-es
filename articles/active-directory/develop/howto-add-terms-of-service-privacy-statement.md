---
title: Condiciones del servicio y declaración de privacidad para aplicaciones | Azure
description: Aprenda a configurar las condiciones del servicio y la declaración de privacidad de las aplicaciones registradas para usar Azure AD.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 05/22/2019
ms.author: ryanwi
ms.reviewer: lenalepa, sureshja
ms.custom: aaddev
ms.openlocfilehash: 73c4931533e3926086320531a0800a572d13808c
ms.sourcegitcommit: c52e50ea04dfb8d4da0e18735477b80cafccc2cf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/08/2020
ms.locfileid: "89535779"
---
# <a name="how-to-configure-terms-of-service-and-privacy-statement-for-an-app"></a>Procedimientos: Configuración de las condiciones del servicio y a la declaración de privacidad para una aplicación

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

| Directrices     | Descripción                           |
|---------------|---------------------------------------|
| Formato        | Dirección URL válida                             |
| Esquemas válidos | HTTP y HTTPS<br/>Se recomienda utilizar HTTPS |
| Longitud máxima    | 2048 caracteres                       |

Ejemplos: `https://myapp.com/terms-of-service` y `https://myapp.com/privacy-statement`

## <a name="adding-links-to-the-terms-of-service-and-privacy-statement"></a>Incorporación de vínculos a las condiciones del servicio y a la declaración de privacidad

Cuando las condiciones del servicio y la declaración de privacidad estén preparados, puede agregar vínculos a estos documentos en la aplicación mediante uno de estos métodos:

* [Mediante Azure Portal](#azure-portal)
* [Con JSON del objeto de aplicación](#app-object-json)
* [Uso de Microsoft Graph API](#msgraph-rest-api)

### <a name="using-the-azure-portal"></a><a name="azure-portal"></a>Mediante Azure Portal
Siga estos pasos en Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y seleccione el inquilino de AzureAD correcto (no B2C).
2. Navegue hasta la sección **Registros de aplicaciones** y seleccione la aplicación.
3. Abra el panel **Personalización de marca**.
4. Rellene los campos **URL de las condiciones del servicio** y **URL de la declaración de privacidad**.
5. Guarde los cambios.

    ![Las propiedades de la aplicación incluyen las direcciones URL de las condiciones del servicio y de la declaración de privacidad.](./media/howto-add-terms-of-service-privacy-statement/azure-portal-terms-service-privacy-statement-urls.png)

### <a name="using-the-app-object-json"></a><a name="app-object-json"></a>Con JSON del objeto de aplicación

Si desea modificar directamente JSON del objeto de aplicación, puede usar el editor de manifiestos en Azure Portal o el Portal de registro de aplicaciones para incluir vínculos en las condiciones del servicio y la declaración de privacidad de la aplicación.

1. Vaya a la sección **Registros de aplicaciones** y seleccione la aplicación.
2. Abra el panel **Manifiesto**.
3. Con Ctrl+F, busque "informationalUrls". Rellene la información.
4. Guarde los cambios.

```json
    "informationalUrls": { 
        "termsOfService": "<your_terms_of_service_url>", 
        "privacy": "<your_privacy_statement_url>" 
    }
```

### <a name="using-the-microsoft-graph-api"></a><a name="msgraph-rest-api"></a>Uso de Microsoft Graph API

Para actualizar mediante programación todas las aplicaciones, puede usar la Microsoft Graph API para la actualización de todas las aplicaciones, de modo que incluyan vínculos en los documentos de los términos del servicio y de la declaración de privacidad.

```
PATCH https://graph.microsoft.com/v1.0/applications/{application id}
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
> * La Microsoft Graph API solo funciona cuando inicia sesión con una cuenta de Azure AD. No se admiten cuentas Microsoft personales.
