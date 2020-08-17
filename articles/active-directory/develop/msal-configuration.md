---
title: Archivo de configuración MSAL de Android | Azure
titleSuffix: Microsoft identity platform
description: Información general sobre el archivo de configuración de la biblioteca de autenticación de Microsoft (MSAL) para Android, que representa la configuración de una aplicación en Azure Active Directory.
services: active-directory
author: shoatman
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: reference
ms.workload: identity
ms.date: 09/12/2019
ms.author: shoatman
ms.custom: aaddev
ms.reviewer: shoatman
ms.openlocfilehash: 3de252b22d7b33e45c3b45e2b6c05e4b33df663d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87027060"
---
# <a name="android-microsoft-authentication-library-configuration-file"></a>Archivo de configuración de la biblioteca de autenticación de Microsoft para Android

La biblioteca de autenticación de Microsoft para Android (MSAL) se suministra con un [archivo JSON de configuración predeterminada](https://github.com/AzureAD/microsoft-authentication-library-for-android/blob/dev/msal/src/main/res/raw/msal_default_config.json) que se personaliza a fin de definir el comportamiento de la aplicación cliente pública para elementos como la autoridad predeterminada, las autoridades que se van a utilizar, etc.

Este artículo le ayudará a comprender los distintos valores del archivo de configuración y a especificar el archivo de configuración que utilizará en la aplicación basada en MSAL.

## <a name="configuration-settings"></a>Parámetros de configuración

### <a name="general-settings"></a>Configuración general

| Propiedad | Tipo de datos | Obligatorio | Notas |
|-----------|------------|-------------|-------|
| `client_id` | String | Sí | Identificador de cliente de la aplicación de la página [Registros de aplicaciones](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) |
| `redirect_uri`   | String | Sí | URI de redireccionamiento de la aplicación de la página [Registros de aplicaciones](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) |
| `authorities` | List\<Authority> | No | Lista de autoridades que la aplicación necesita |
| `authorization_user_agent` | AuthorizationAgent (enum) | No | Valores posibles: `DEFAULT`, `BROWSER`, `WEBVIEW` |
| `http` | HttpConfiguration | No | Configure `HttpUrlConnection` `connect_timeout` y `read_timeout`. |
| `logging` | LoggingConfiguration | No | Especifica el nivel de detalle del registro. Entre las configuraciones opcionales se incluyen: `pii_enabled`, que toma un valor booleano, y `log_level`, que toma `ERROR`, `WARNING`, `INFO` o `VERBOSE`. |

### <a name="client_id"></a>client_id

Identificador de cliente o identificador de aplicación creado al registrar la aplicación.

### <a name="redirect_uri"></a>redirect_uri

URI de redireccionamiento registrado al registrar la aplicación. Si el URI de redireccionamiento es a una aplicación de agente, consulte [URI de redireccionamiento para aplicaciones cliente públicas](msal-client-application-configuration.md#redirect-uri-for-public-client-apps) a fin de asegurarse de que está usando el formato de URI de redireccionamiento correcto para la aplicación de agente.

### <a name="authorities"></a>autoridades

Lista de autoridades que conoce y en las que confía. Además de las autoridades aquí mencionadas, MSAL también consulta a Microsoft para obtener una lista de las nubes y autoridades que conoce Microsoft. En esta lista de autoridades, especifique el tipo de autoridad y cualquier parámetro opcional adicional, como `"audience"`, que se debe adaptar a la audiencia de la aplicación en función del registro de la aplicación. A continuación se muestra una lista de autoridades de ejemplo:

```javascript
// Example AzureAD and Personal Microsoft Account
{
    "type": "AAD",
    "audience": {
        "type": "AzureADandPersonalMicrosoftAccount"
    },
    "default": true // Indicates that this is the default to use if not provided as part of the acquireToken or acquireTokenSilent call
},
// Example AzureAD My Organization
{
    "type": "AAD",
    "audience": {
        "type": "AzureADMyOrg",
        "tenant_id": "contoso.com" // Provide your specific tenant ID here
    }
},
// Example AzureAD Multiple Organizations
{
    "type": "AAD",
    "audience": {
        "type": "AzureADMultipleOrgs"
    }
},
//Example PersonalMicrosoftAccount
{
    "type": "AAD",
    "audience": {
        "type": "PersonalMicrosoftAccount"
    }
}
```

#### <a name="map-aad-authority--audience-to-microsoft-identity-platform-endpoints"></a>Asignación de una autoridad de AAD y audiencia a puntos de conexión de la Plataforma de identidad de Microsoft

| Tipo | Público | Id. de inquilino | Authority_Url | Punto de conexión resultante | Notas |
|------|------------|------------|----------------|----------------------|---------|
| AAD | AzureADandPersonalMicrosoftAccount | | | `https://login.microsoftonline.com/common` | `common` es un alias de inquilino de donde se encuentra la cuenta. Como, por ejemplo, un inquilino de Azure Active Directory específico o el sistema de cuentas de Microsoft. |
| AAD | AzureADMyOrg | contoso.com | | `https://login.microsoftonline.com/contoso.com` | Solo las cuentas presentes en contoso.com pueden adquirir un token. Cualquier dominio comprobado, o el GUID del inquilino, se puede usar como identificador de inquilino. |
| AAD | AzureADMultipleOrgs | | | `https://login.microsoftonline.com/organizations` | Solo se pueden utilizar cuentas de Azure Active Directory con este punto de conexión. Las cuentas de Microsoft pueden ser miembros de organizaciones. Para adquirir un token mediante un cuenta de Microsoft para un recurso de una organización, especifique el inquilino de la organización del que desea el token. |
| AAD | PersonalMicrosoftAccount | | | `https://login.microsoftonline.com/consumers` | Solo las cuentas de Microsoft pueden usar este punto de conexión. |
| B2C | | | Consulte Punto de conexión resultante | `https://login.microsoftonline.com/tfp/contoso.onmicrosoft.com/B2C_1_SISOPolicy/` | Solo las cuentas presentes en el inquilino contoso.onmicrosoft.com pueden adquirir un token. En este ejemplo, la directiva de B2C forma parte de la ruta de la dirección URL de la autoridad. |

> [!NOTE]
> No se puede habilitar ni deshabilitar la validación de la autoridad en MSAL.
> Las autoridades son conocidas por usted como el desarrollador tal como se especifica mediante la configuración o son conocidas por Microsoft mediante metadatos.
> Si MSAL recibe una solicitud de un token para una autoridad desconocida, se produce una `MsalClientException` de tipo `UnknownAuthority`.

#### <a name="authority-properties"></a>Propiedades de la autoridad

| Propiedad | Tipo de datos  | Obligatorio | Notas |
|-----------|-------------|-----------|--------|
| `type` | String | Sí | Refleja la audiencia o el tipo de cuenta a la que se dirige la aplicación. Valores posibles: `AAD`, `B2C` |
| `audience` | Object | No | Solo se aplica cuando type=`AAD`. Especifica la identidad a la que se dirige la aplicación. Utilice el valor del registro de la aplicación |
| `authority_url` | String | Sí | Solo es necesario cuando type=`B2C`. Especifica la dirección URL de la autoridad o la directiva que debe usar la aplicación.  |
| `default` | boolean | Sí | Se requiere un único `"default":true` cuando se especifica una o varias autoridades. |

#### <a name="audience-properties"></a>Propiedades de la audiencia

| Propiedad | Tipo de datos  | Obligatorio | Notas |
|-----------|-------------|------------|-------|
| `type` | String | Sí | Especifica la audiencia a la que la aplicación quiere dirigirse. Valores posibles: `AzureADandPersonalMicrosoftAccount`, `PersonalMicrosoftAccount`, `AzureADMultipleOrgs`, `AzureADMyOrg` |
| `tenant_id` | String | Sí | Solo es necesario cuando `"type":"AzureADMyOrg"`. Opcional para otros valores `type`. Puede ser un dominio de inquilino como `contoso.com` o un identificador de inquilino como `72f988bf-86f1-41af-91ab-2d7cd011db46`. |

### <a name="authorization_user_agent"></a>authorization_user_agent

Indica si se va a usar un WebView incrustado, o el explorador predeterminado en el dispositivo, al iniciar sesión en una cuenta o al autorizar el acceso a un recurso.

Valores posibles:
- `DEFAULT`: Prefiere el explorador del sistema. Usa el WebView incrustado si no hay un explorador disponible en el dispositivo.
- `WEBVIEW`: Utiliza el objeto WebView insertado.
- `BROWSER`: Usa el explorador predeterminado en el dispositivo.

### <a name="multiple_clouds_supported"></a>multiple_clouds_supported

Para clientes que admiten varias nubes nacionales, especifique `true`. La Plataforma de identidad de Microsoft se redirigirá automáticamente a la nube nacional correcta durante la autorización y el canje de tokens. Puede determinar la nube nacional de la cuenta con la que se ha iniciado sesión examinando la autoridad asociada con `AuthenticationResult`. Tenga en cuenta que `AuthenticationResult` no proporciona la dirección del punto de conexión nacional específica de la nube del recurso para el que solicita un token.

### <a name="broker_redirect_uri_registered"></a>broker_redirect_uri_registered

Valor booleano que indica si está usando un URI de redireccionamiento de agente compatible con el agente de identidad de Microsoft. Establézcalo en `false` si no desea usar el agente en la aplicación.

Si utiliza la autoridad de AAD con la audiencia establecida en `"MicrosoftPersonalAccount"`, no se usará el agente.

### <a name="http"></a>http

Configure los valores globales para los tiempos de espera de HTTP, como:

| Propiedad | Tipo de datos | Obligatorio | Notas |
| ---------|-----------|------------|--------|
| `connect_timeout` | int | No | Tiempo en milisegundos |
| `read_timeout` | int | No | Tiempo en milisegundos |

### <a name="logging"></a>logging

La siguiente configuración global es para el registro:

| Propiedad | Tipo de datos  | Obligatorio | Notas |
| ----------|-------------|-----------|---------|
| `pii_enabled`  | boolean | No | Si se van a emitir datos personales |
| `log_level`   | string | No | Qué mensajes de registro se van a generar. Entre los niveles de registro se incluyen `ERROR`,`WARNING`,`INFO` y `VERBOSE`. |
| `logcat_enabled` | boolean | No | Indica si se va a generar el catálogo de registros, además de la interfaz de registro |

### <a name="account_mode"></a>account_mode

Especifica el número de cuentas que se pueden usar en la aplicación a la vez. Los valores posibles son:

- `MULTIPLE` (Valor predeterminado)
- `SINGLE`

Si se construye `PublicClientApplication` mediante un modo de cuenta que no coincide con esta configuración, se producirá una excepción.

Para más información sobre las diferencias entre una o varias cuentas, consulte [Single and multiple account apps](single-multi-account.md) (Aplicaciones de una o varias cuentas).

### <a name="browser_safelist"></a>browser_safelist

Lista de exploradores permitidos que son compatibles con MSAL. Estos exploradores controlan correctamente los redireccionamientos a intentos personalizados. Puede agregar elementos a esta lista. El valor predeterminado se proporciona en la configuración predeterminada que se muestra a continuación.
``
## <a name="the-default-msal-configuration-file"></a>Archivo de configuración de MSAL predeterminado

A continuación se muestra la configuración predeterminada de MSAL que se incluye con MSAL. Puede ver la versión más reciente en [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-android/blob/dev/msal/src/main/res/raw/msal_default_config.json).

Esta configuración se complementa con los valores que proporcione. Los valores que proporcione invalidan los valores predeterminados.

```javascript
{
  "authorities": [
    {
      "type": "AAD",
      "audience": {
        "type": "AzureADandPersonalMicrosoftAccount"
      },
      "default": true
    }
  ],
  "authorization_user_agent": "DEFAULT",
  "multiple_clouds_supported": false,
  "broker_redirect_uri_registered": false,
  "http": {
    "connect_timeout": 10000,
    "read_timeout": 30000
  },
  "logging": {
    "pii_enabled": false,
    "log_level": "WARNING",
    "logcat_enabled": false
  },
  "shared_device_mode_supported": false,
  "account_mode": "MULTIPLE",
  "browser_safelist": [
    {
      "browser_package_name": "com.android.chrome",
      "browser_signature_hashes": [
        "7fmduHKTdHHrlMvldlEqAIlSfii1tl35bxj1OXN5Ve8c4lU6URVu4xtSHc3BVZxS6WWJnxMDhIfQN0N0K2NDJg=="
      ],
      "browser_use_customTab" : true,
      "browser_version_lower_bound": "45"
    },
    {
      "browser_package_name": "com.android.chrome",
      "browser_signature_hashes": [
        "7fmduHKTdHHrlMvldlEqAIlSfii1tl35bxj1OXN5Ve8c4lU6URVu4xtSHc3BVZxS6WWJnxMDhIfQN0N0K2NDJg=="
      ],
      "browser_use_customTab" : false
    },
    {
      "browser_package_name": "org.mozilla.firefox",
      "browser_signature_hashes": [
        "2gCe6pR_AO_Q2Vu8Iep-4AsiKNnUHQxu0FaDHO_qa178GByKybdT_BuE8_dYk99G5Uvx_gdONXAOO2EaXidpVQ=="
      ],
      "browser_use_customTab" : false
    },
    {
      "browser_package_name": "org.mozilla.firefox",
      "browser_signature_hashes": [
        "2gCe6pR_AO_Q2Vu8Iep-4AsiKNnUHQxu0FaDHO_qa178GByKybdT_BuE8_dYk99G5Uvx_gdONXAOO2EaXidpVQ=="
      ],
      "browser_use_customTab" : true,
      "browser_version_lower_bound": "57"
    },
    {
      "browser_package_name": "com.sec.android.app.sbrowser",
      "browser_signature_hashes": [
        "ABi2fbt8vkzj7SJ8aD5jc4xJFTDFntdkMrYXL3itsvqY1QIw-dZozdop5rgKNxjbrQAd5nntAGpgh9w84O1Xgg=="
      ],
      "browser_use_customTab" : true,
      "browser_version_lower_bound": "4.0"
    },
    {
      "browser_package_name": "com.sec.android.app.sbrowser",
      "browser_signature_hashes": [
        "ABi2fbt8vkzj7SJ8aD5jc4xJFTDFntdkMrYXL3itsvqY1QIw-dZozdop5rgKNxjbrQAd5nntAGpgh9w84O1Xgg=="
      ],
      "browser_use_customTab" : false
    },
    {
      "browser_package_name": "com.cloudmosa.puffinFree",
      "browser_signature_hashes": [
        "1WqG8SoK2WvE4NTYgr2550TRhjhxT-7DWxu6C_o6GrOLK6xzG67Hq7GCGDjkAFRCOChlo2XUUglLRAYu3Mn8Ag=="
      ],
      "browser_use_customTab" : false
    },
    {
      "browser_package_name": "com.duckduckgo.mobile.android",
      "browser_signature_hashes": [
        "S5Av4cfEycCvIvKPpKGjyCuAE5gZ8y60-knFfGkAEIZWPr9lU5kA7iOAlSZxaJei08s0ruDvuEzFYlmH-jAi4Q=="
      ],
      "browser_use_customTab" : false
    },
    {
      "browser_package_name": "com.explore.web.browser",
      "browser_signature_hashes": [
        "BzDzBVSAwah8f_A0MYJCPOkt0eb7WcIEw6Udn7VLcizjoU3wxAzVisCm6bW7uTs4WpMfBEJYf0nDgzTYvYHCag=="
      ],
      "browser_use_customTab" : false
    },

    {
      "browser_package_name": "com.ksmobile.cb",
      "browser_signature_hashes": [
        "lFDYx1Rwc7_XUn4KlfQk2klXLufRyuGHLa3a7rNjqQMkMaxZueQfxukVTvA7yKKp3Md3XUeeDSWGIZcRy7nouw=="
      ],
      "browser_use_customTab" : false
    },

    {
      "browser_package_name": "com.microsoft.emmx",
      "browser_signature_hashes": [
        "Ivy-Rk6ztai_IudfbyUrSHugzRqAtHWslFvHT0PTvLMsEKLUIgv7ZZbVxygWy_M5mOPpfjZrd3vOx3t-cA6fVQ=="
      ],
      "browser_use_customTab" : false
    },

    {
      "browser_package_name": "com.opera.browser",
      "browser_signature_hashes": [
        "FIJ3IIeqB7V0qHpRNEpYNkhEGA_eJaf7ntca-Oa_6Feev3UkgnpguTNV31JdAmpEFPGNPo0RHqdlU0k-3jWJWw=="
      ],
      "browser_use_customTab" : false
    },

    {
      "browser_package_name": "com.opera.mini.native",
      "browser_signature_hashes": [
        "TOTyHs086iGIEdxrX_24aAewTZxV7Wbi6niS2ZrpPhLkjuZPAh1c3NQ_U4Lx1KdgyhQE4BiS36MIfP6LbmmUYQ=="
      ],
      "browser_use_customTab" : false
    },

    {
      "browser_package_name": "mobi.mgeek.TunnyBrowser",
      "browser_signature_hashes": [
        "RMVoXuK1sfJZuGZ8onG1yhMc-sKiAV2NiB_GZfdNlN8XJ78XEE2wPM6LnQiyltF25GkHiPN2iKQiGwaO2bkyyQ=="
      ],
      "browser_use_customTab" : false
    },

    {
      "browser_package_name": "org.mozilla.focus",
      "browser_signature_hashes": [
        "L72dT-stFqomSY7sYySrgBJ3VYKbipMZapmUXfTZNqOzN_dekT5wdBACJkpz0C6P0yx5EmZ5IciI93Q0hq0oYA=="
      ],
      "browser_use_customTab" : false
    }
  ]
}
```
## <a name="example-basic-configuration"></a>Ejemplo de configuración básica

En el ejemplo siguiente se muestra una configuración básica que especifica el identificador de cliente, el URI de redireccionamiento, si se ha registrado un redireccionamiento de agente y una lista de autoridades.

```javascript
{
  "client_id" : "4b0db8c2-9f26-4417-8bde-3f0e3656f8e0",
  "redirect_uri" : "msauth://com.microsoft.identity.client.sample.local/1wIqXSqBj7w%2Bh11ZifsnqwgyKrY%3D",
  "broker_redirect_uri_registered": true,
  "authorities" : [
    {
      "type": "AAD",
      "audience": {
        "type": "AzureADandPersonalMicrosoftAccount"
      }
      "default": true
    }
  ]
}
```

## <a name="how-to-use-a-configuration-file"></a>Uso de un archivo de configuración

1. Cree un archivo de configuración. Se recomienda crear un archivo de configuración predeterminado en `res/raw/auth_config.json`. Pero puede colocarlo en cualquier lugar que desee.
2. Indique a MSAL dónde debe buscar la configuración al construir `PublicClientApplication`. Por ejemplo:

   ```java
   //On Worker Thread
   IMultipleAccountPublicClientApplication sampleApp = null; 
   sampleApp = new PublicClientApplication.createMultipleAccountPublicClientApplication(getApplicationContext(), R.raw.auth_config);
   ```
