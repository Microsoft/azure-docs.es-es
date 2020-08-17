---
title: Diferencias entre MSAL.js y ADAL.js | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre las diferencias entre la Biblioteca de autenticación de Microsoft para JavaScript (MSAL.js) y la Biblioteca de autenticación de Active Directory para JavaScript (ADAL.js) y cómo elegir cuál usar.
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 04/10/2019
ms.author: nacanuma
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 5d91514af9aea3dd7ea13b94681fbb27c53772fe
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88120905"
---
# <a name="differences-between-msal-js-and-adal-js"></a>Diferencias entre MSAL JS y ADAL JS

Tanto la Biblioteca de autenticación de Microsoft para JavaScript (MSAL.js) como la Biblioteca de autenticación de Active Directory (ADAL.js) se usan para autenticar entidades de Azure AD y solicitar tokens a Azure AD. Hasta ahora, la mayoría de los desarrolladores ha trabajado con Azure AD para que los desarrolladores (v1.0) autentiquen identidades de Azure AD (cuentas profesionales y educativas) mediante la solicitud de tokens con ADAL. Ahora, mediante MSAL.js, es posible autenticar a un conjunto más amplio de identidades de Microsoft (identidades de Azure AD y cuentas Microsoft, así como cuentas sociales y locales a través de Azure AD B2C) a través de la plataforma de identidad de Microsoft (v2.0).

En este artículo se describe cómo elegir entre la Biblioteca de autenticación de Microsoft para JavaScript (MSAL.js) y la Biblioteca de autenticación de Active Directory para JavaScript (ADAL.js) y se comparan ambas bibliotecas.

## <a name="choosing-between-adaljs-and-msaljs"></a>Elección entre ADAL.js y MSAL.js

En la mayoría de los casos deseará usar la plataforma de identidad de Microsoft y MSAL.js, que es la última generación de bibliotecas de autenticación de Microsoft. Mediante MSAL.js, adquiere tokens para los usuarios que inician sesión en su aplicación con Azure AD (cuentas profesionales y educativas), cuentas (personales) Microsoft (MSA) o Azure AD B2C.

Si ya conoce el punto de conexión v1.0 (y ADAL.js), es posible que quiera consultar [¿Qué hay diferente en el punto de conexión v2.0?](../azuread-dev/azure-ad-endpoint-comparison.md).

Sin embargo, aun así deberá usar ADAL.js si la aplicación necesita iniciar sesión en los usuarios con versiones anteriores de [Servicios de federación de Active Directory (AD FS)](/windows-server/identity/active-directory-federation-services).

## <a name="key-differences-in-authentication-with-msaljs"></a>Diferencias clave en la autenticación con MSAL.js

### <a name="core-api"></a>Core API

* ADAL.js usa [AuthenticationContext](https://github.com/AzureAD/azure-activedirectory-library-for-js/wiki/Config-authentication-context#authenticationcontext) como representación de una instancia de la conexión de la aplicación con el servidor de autorización o el proveedor de identidades a través de una dirección URL de la autoridad. Por el contrario, MSAL.js API se diseñó en torno a una aplicación cliente de agente de usuario (una forma de aplicación cliente pública en la que el código cliente se ejecuta en un agente de usuario, como un explorador web). Proporciona la clase `UserAgentApplication` para representar una instancia del contexto de autenticación de la aplicación con el servidor de autorización. Para más detalles, consulte el artículo sobre la [inicialización con MSAL.js](msal-js-initializing-client-applications.md).

* En ADAL.js, los métodos para adquirir tokens están asociados con una sola entidad establecida en `AuthenticationContext`. En MSAL.js, las solicitudes de adquisición de token pueden tener valores de autoridad distintos a los que se enviaron en `UserAgentApplication`. Esto permite que MSAL.js adquiera y almacene en caché tokens por separado para varios inquilinos y cuentas de usuario en la misma aplicación.

* El método para adquirir y renovar tokens de manera silenciosa sin solicitar nada a los usuarios se denomina `acquireToken` en ADAL.js. En MSAL.js, este método se denomina `acquireTokenSilent` para describir mejor esta funcionalidad.

### <a name="authority-value-common"></a>Valor de autoridad `common`

En la versión 1.0, si se usa la autoridad `https://login.microsoftonline.com/common` se permitirá que los usuarios inicien sesión con cualquier cuenta de Azure AD (para cualquier organización).

En la versión 2.0, si se usa la autoridad `https://login.microsoftonline.com/common`, se permitirá que los usuarios inicien sesión con cualquier cuenta de organización o con una cuenta personal de Microsoft (MSA). Para restringir el inicio de sesión solo a cuentas de Azure AD (el mismo comportamiento que con ADAL.js), debe usar `https://login.microsoftonline.com/organizations`. Si necesita conocer detalles, consulte la opción de configuración `authority` en el artículo sobre la [inicialización con MSAL.js](msal-js-initializing-client-applications.md).

### <a name="scopes-for-acquiring-tokens"></a>Ámbitos para la adquisición de tokens
* Ámbito en lugar de parámetro de recurso en solicitudes de autenticación para adquirir tokens

    El protocolo de la versión v2.0 usa ámbitos en lugar de recursos en las solicitudes. Es decir, cuando la aplicación tenga que solicitar tokens con permisos para un recurso como MS Graph, la diferencia en los valores pasados a los métodos de biblioteca son los siguientes:

    v1.0: resource = https\://graph.microsoft.com

    v2.0: scope = https\://graph.microsoft.com/User.Read

    Puede solicitar ámbitos para cualquier API de recurso con el URI de la API con este formato: appidURI/scope, por ejemplo: https:\//mi-inquilino.onmicrosoft.com/myapi/api.read

    Solo para MS Graph API, el valor de ámbito `user.read` se asigna a https:\//graph.microsoft.com/User.Read y se puede usar indistintamente.

    ```javascript
    var request = {
        scopes = ["User.Read"];
    };

    acquireTokenPopup(request);   
    ```

* Ámbitos dinámicos para consentimiento incremental.

    Cuando compilaba aplicaciones con la versión 1.0, era necesario registrar el conjunto completo de permisos (ámbitos estáticos) que requería la aplicación para que el usuario diera su consentimiento en el momento del inicio de sesión. En la versión 2.0, puede usar el parámetro de ámbito para solicitar los permisos en el momento en que lo desee. Se denominan "ámbitos dinámicos". Esto permite que el usuario dé su consentimiento incremental a los ámbitos. Por lo tanto, si al inicio solo quería que el usuario iniciara sesión en la aplicación y no necesita ningún tipo de acceso, puede hacerlo. Si posteriormente necesita poder leer el calendario del usuario, podrá solicitar el ámbito de calendario en los métodos acquireToken y obtener el consentimiento del usuario. Por ejemplo:

    ```javascript
    var request = {
        scopes = ["https://graph.microsoft.com/User.Read", "https://graph.microsoft.com/Calendar.Read"];
    };

    acquireTokenPopup(request);   
    ```

* Ámbitos para las API de la versión 1.0

    Al obtener tokens para las API de la versión 1.0 con MSAL.js, puede solicitar todos los ámbitos estáticos registrados en la API si anexa `.default` al URI del identificador de la aplicación de la API como ámbito. Por ejemplo:

    ```javascript
    var request = {
        scopes = [ appidURI + "/.default"];
    };

    acquireTokenPopup(request);
    ```

## <a name="next-steps"></a>Pasos siguientes
Para más información, consulte la [comparación entre la versión 1 y la versión 2](../azuread-dev/azure-ad-endpoint-comparison.md).