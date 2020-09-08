---
title: Microsoft Graph API para Azure Active Directory Identity Protection
description: Obtenga información sobre cómo realizar una consulta a Microsoft Graph para obtener una lista de detecciones de riesgo e información asociada desde Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: how-to
ms.date: 07/14/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 86d88f841f76b367e83f0ae6b81e604e1b7f3e4b
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88950126"
---
# <a name="get-started-with-azure-active-directory-identity-protection-and-microsoft-graph"></a>Introducción a Azure Active Directory Identity Protection y Microsoft Graph

Microsoft Graph es el punto de conexión de API unificada de Microsoft y donde se encuentran las API de [Azure Active Directory Identity Protection](./overview-identity-protection.md). Hay tres API que exponen información acerca de los usuarios e inicios de sesión con riesgo. La primera API, **riskDetection**, permite consultar Microsoft Graph para obtener una lista de las detecciones de riesgo vinculado de usuario e inicio de sesión y la información asociada sobre la detección. La segunda API, **riskyUsers**, permite consultar Microsoft Graph, para obtener información acerca de qué usuarios ha detectado Identity Protection como un riesgo. La tercera API, **signIn**, permite consultar Microsoft Graph para obtener información acerca de los inicios de sesión de Azure AD con propiedades concretas relacionadas con el estado, detalle y nivel del riesgo. 

En este artículo se le introduce a la conexión a Microsoft Graph y a la consulta de estas API. Para obtener una introducción más detallada, ver toda la documentación y acceder al Probador de Graph, diríjase al [sitio web de Microsoft Graph](https://graph.microsoft.io/) o consulte la documentación de referencia específica de estas API:

* [riskDetection API](/graph/api/resources/riskdetection?view=graph-rest-v1.0)
* [riskyUsers API](/graph/api/resources/riskyuser?view=graph-rest-v1.0)
* [signIn API](/graph/api/resources/signin?view=graph-rest-v1.0)

## <a name="connect-to-microsoft-graph"></a>Conexión con Microsoft Graph

Hay cuatro pasos para acceder a los datos de Identity Protection a través de Microsoft Graph:

- [Recuperación del nombre de dominio](#retrieve-your-domain-name)
- [Creación de un nuevo registro de aplicaciones](#create-a-new-app-registration)
- [Configuración de permisos de API](#configure-api-permissions)
- [Configuración de una credencial válida](#configure-a-valid-credential)

### <a name="retrieve-your-domain-name"></a>Recuperación del nombre de dominio 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).  
1. Vaya a **Azure Active Directory** > **Nombres de dominio personalizados**. 
1. Tome nota del dominio `.onmicrosoft.com`, necesitará esta información en un paso posterior.

### <a name="create-a-new-app-registration"></a>Creación de un registro de aplicaciones

1. En Azure Portal, vaya a **Azure Active Directory** > **Registros de aplicaciones**.
1. Seleccione **Nuevo registro**.
1. En la página **Crear**, realice los pasos siguientes:
   1. En el cuadro de texto **Nombre**, escriba el nombre de la aplicación (por ejemplo: API Risk Detection de Azure AD).
   1. En **Tipos de cuenta compatibles**, seleccione el tipo de cuentas que utilizarán las API.
   1. Seleccione **Registrar**.
1. Copie el **id. de la aplicación**.

### <a name="configure-api-permissions"></a>Configuración de permisos de API

1. Desde la **Aplicación** que creó, seleccione **Permisos de API**.
1. En la página **Permisos configurados**, en la barra de herramientas de la parte superior, haga clic en **Agregar un permiso**.
1. En la página **Agregar acceso de API**, haga clic en **Seleccionar una API**.
1. En la página **Seleccionar una API**, seleccione **Microsoft Graph** y, a continuación, haga clic en **Seleccionar**.
1. En la página **Solicitud de permisos de API**. 
   1. Seleccione **Permisos de aplicación**.
   1. Active las casillas junto a `IdentityRiskEvent.Read.All` y `IdentityRiskyUser.Read.All`.
   1. Seleccione **Agregar permisos**.
1. Seleccione **Conceder consentimiento del administrador para el dominio**. 

### <a name="configure-a-valid-credential"></a>Configuración de una credencial válida

1. Desde la **Aplicación** que creó, seleccione **Certificados y secretos**.
1. En **Secretos de cliente**, seleccione **Nuevo secreto de cliente**.
   1. Escriba una **Descripción** del secreto de cliente y establezca el período de expiración de acuerdo con las directivas de la organización.
   1. Seleccione **Agregar**.

   > [!NOTE]
   > Si pierde esta clave, tendrá que volver a esta sección y crear una nueva. Guarde esta clave como un secreto: cualquier persona que la tenga accederá a sus datos.

## <a name="authenticate-to-microsoft-graph-and-query-the-identity-risk-detections-api"></a>Autenticación en Microsoft Graph y consulta a Identity Risk Detections API

En este momento, debe tener:

- El nombre de dominio del inquilino
- El identificador la aplicación (cliente) 
- El secreto de cliente o certificado 

Para autenticarse, envíe una solicitud post a `https://login.microsoft.com` con los siguientes parámetros en el cuerpo:

- grant_type: “**client_credentials**”
- resource: `https://graph.microsoft.com`
- client_id: \<your client ID\>
- client_secret: \<your key\>

Si esta solicitud se completa correctamente, devuelve un token de autenticación.  
Para llamar a la API, cree un encabezado con el parámetro siguiente:

```
`Authorization`="<token_type> <access_token>"
```

Al autenticar, puede encontrar el tipo de token y el token de acceso en el token devuelto.

Envíe este encabezado como una solicitud a la siguiente dirección URL de la API: `https://graph.microsoft.com/v1.0/identityProtection/riskDetections`

La respuesta, si se ha realizado correctamente, es una colección de detecciones de riesgo de identidad y datos asociados en formato JSON de OData, que se pueden analizar y tratar como convenga.

### <a name="sample"></a>Muestra

Este es el código de ejemplo para autenticar y llamar a la API mediante PowerShell.  
Solo tiene que agregar el identificador de cliente, la clave secreta y el dominio del inquilino.

```PowerShell
    $ClientID       = "<your client ID here>"        # Should be a ~36 hex character string; insert your info here
    $ClientSecret   = "<your client secret here>"    # Should be a ~44 character string; insert your info here
    $tenantdomain   = "<your tenant domain here>"    # For example, contoso.onmicrosoft.com

    $loginURL       = "https://login.microsoft.com"
    $resource       = "https://graph.microsoft.com"

    $body       = @{grant_type="client_credentials";resource=$resource;client_id=$ClientID;client_secret=$ClientSecret}
    $oauth      = Invoke-RestMethod -Method Post -Uri $loginURL/$tenantdomain/oauth2/token?api-version=1.0 -Body $body

    Write-Output $oauth

    if ($oauth.access_token -ne $null) {
        $headerParams = @{'Authorization'="$($oauth.token_type) $($oauth.access_token)"}

        $url = "https://graph.microsoft.com/v1.0/identityProtection/riskDetections"
        Write-Output $url

        $myReport = (Invoke-WebRequest -UseBasicParsing -Headers $headerParams -Uri $url)

        foreach ($event in ($myReport.Content | ConvertFrom-Json).value) {
            Write-Output $event
        }

    } else {
        Write-Host "ERROR: No Access Token"
    } 
```

## <a name="query-the-apis"></a>Consulta de las API

Estas tres API ofrecen una gran cantidad de oportunidades para recuperar información acerca de los usuarios e inicios de sesión de riesgo en su organización. A continuación se muestran algunos casos de uso comunes de estas API y las solicitudes de ejemplo asociadas. Para ejecutar puede usar el código de ejemplo anterior, o el [Probador de Graph](https://developer.microsoft.com/graph/graph-explorer).

### <a name="get-all-of-the-offline-risk-detections-riskdetection-api"></a>Obtener todas las detecciones de riesgo sin conexión (riskDetection API)

Con las directivas de riesgo de inicio de sesión de Identity Protection, puede aplicar condiciones cuando se detecta un riesgo en tiempo real. Pero ¿qué ocurre con las detecciones que se detectan sin conexión? Para comprender qué detecciones se han producido sin conexión y, por lo tanto, no habrían desencadenado la directiva de riesgo de inicio de sesión, puede consultar riskDetection API.

```
GET https://graph.microsoft.com/v1.0/identityProtection/riskDetections?$filter=detectionTimingType eq 'offline'
```

### <a name="get-all-of-the-users-who-successfully-passed-an-mfa-challenge-triggered-by-risky-sign-ins-policy-riskyusers-api"></a>Obtención de todos los usuarios que han superado satisfactoriamente un desafío de MFA desencadenado por la directiva de inicios de sesión de riesgo (riskyUsers API)

Para conocer el impacto que las directivas basadas en riesgo de Identity Protection tienen en su organización, puede consultar todos los usuarios que han superado satisfactoriamente un desafío de MFA desencadenado por una directiva de inicios de sesión de riesgo. Esta información puede ayudarle a saber qué usuarios puede haber detectado Identity Protection falsamente como un riesgo y cuáles de los usuarios legítimos pueden estar realizando acciones que la IA considera de riesgo.

```
GET https://graph.microsoft.com/v1.0/identityProtection/riskyUsers?$filter=riskDetail eq 'userPassedMFADrivenByRiskBasedPolicy'
```

## <a name="next-steps"></a>Pasos siguientes

Enhorabuena, acaba de hacer la primera llamada a Microsoft Graph.  
Ahora puede consultar las detecciones de riesgo de identidad y utilizar los datos cuando lo estime necesario.

Para obtener más información sobre Microsoft Graph y cómo crear aplicaciones con Graph API, consulte la [documentación](/graph/overview) y muchos más detalles en el [sitio web de Microsoft Graph](https://developer.microsoft.com/graph). 

Para obtener información relacionada, consulte:

- [Azure Active Directory Identity Protection](./overview-identity-protection.md)
- [Tipos de detecciones de riesgo detectadas por Azure Active Directory Identity Protection](./overview-identity-protection.md)
- [Microsoft Graph](https://developer.microsoft.com/graph/)
- [Overview of Microsoft Graph (Información general de Microsoft Graph)](https://developer.microsoft.com/graph/docs)
- [Azure AD Identity Protection Service Root (Raíz del servicio de Azure AD Identity Protection)](/graph/api/resources/identityprotectionroot?view=graph-rest-1.0)