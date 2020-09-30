---
title: Uso de MSAL en una aplicación en la nube nacional | Azure
titleSuffix: Microsoft identity platform
description: La biblioteca de autenticación de Microsoft (MSAL) permite a los desarrolladores de aplicaciones adquirir tokens para llamar a las API web protegidas. Estas API web pueden ser Microsoft Graph, otras API de Microsoft, API web de terceros o su propia API web. MSAL admite varias plataformas y arquitecturas de aplicación.
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/22/2019
ms.author: negoe
ms.reviewer: nacanuma
ms.custom: aaddev
ms.openlocfilehash: 0b54a8227594a81c17dcaaaaa6c599d70217c498
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90705867"
---
# <a name="use-msal-in-a-national-cloud-environment"></a>Uso de MSAL en un entorno de nube nacional

Las [nubes nacionales](authentication-national-cloud.md), también conocidas como nubes soberanas, son instancias físicamente aisladas de Azure. Estas regiones de Azure ayudan a asegurarse de que se cumplan los requisitos de residencia, soberanía y cumplimiento normativo de los datos dentro de fronteras geográficas.

Además de la nube mundial de Microsoft, la biblioteca de autenticación de Microsoft (MSAL) permite a los desarrolladores de aplicaciones en nubes nacionales adquirir tokens con el fin de autenticar e invocar API de web protegidas. Estas API web pueden Microsoft Graph u otras API de Microsoft.

Incluida la nube mundial, Azure Active Directory (Azure AD) se implementa en las siguientes nubes nacionales:  

- Azure Government
- Azure China 21Vianet
- Azure Alemania

En esta guía se muestra cómo iniciar sesión en cuentas profesionales y educativas, obtener un token de acceso y llamar a la API de Microsoft Graph en el entorno de la [nube de Azure Government](https://azure.microsoft.com/global-infrastructure/government/).

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar, asegúrese de que cumpla estos requisitos previos.

### <a name="choose-the-appropriate-identities"></a>Elección de las identidades adecuadas

Las aplicaciones de [Azure Government](../../azure-government/index.yml) pueden usar identidades de Azure AD Government e identidades pública de Azure AD para autenticar a los usuarios. Dado que puede usar cualquiera de estas identidades, debe decidir qué punto de conexión de autoridad debe elegir para su escenario:

- Azure AD público: se suele usar si la organización ya tiene un inquilino público de Azure AD para admitir Microsoft 365 (público o GCC) u otra aplicación.
- Azure AD Government: se suele usar si la organización ya tiene un inquilino de Azure AD Government para admitir Office 365 (GCC High o DoD) o está creando un nuevo inquilino en Azure AD Government.

Una vez decidido, debe tener en cuenta también dónde se realiza el registro de la aplicación. Si elige identidades de Azure AD público para la aplicación de Azure Government, debe registrar la aplicación en el inquilino de Azure AD público.

### <a name="get-an-azure-government-subscription"></a>Obtener una suscripción a Azure Government

Para obtener una suscripción a Azure Government, consulte [Administración y conexión a su suscripción en Azure Government](../../azure-government/documentation-government-manage-subscriptions.md).

Si no tiene una suscripción a Azure Government, cree una [cuenta gratuita](https://azure.microsoft.com/global-infrastructure/government/request/) antes de comenzar.

Para obtener más información sobre el uso de una nube nacional con un lenguaje de programación determinado, elija la pestaña que coincida con su lenguaje:

## <a name="net"></a>[.NET](#tab/donet)

Puede usar MSAL.NET para iniciar la sesión de los usuarios, adquirir tokens y llamar a la API de Microsoft Graph en nubes nacionales.

En los siguientes tutoriales se muestra cómo compilar una aplicación web MVC de .NET Core 2.2. La aplicación usa OpenID Connect para iniciar la sesión de los usuarios con una cuenta profesional y educativa en una organización que pertenece a una nube nacional.

- Para iniciar la sesión de los usuarios y adquirir tokens, siga este tutorial: [Cree una aplicación web de ASP.NET Core que permita iniciar la sesión de usuarios en nubes soberanas con la Plataforma de identidad de Microsoft](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-4-Sovereign#build-an-aspnet-core-web-app-signing-in-users-in-sovereign-clouds-with-the-microsoft-identity-platform).
- Para llamar a la API de Microsoft Graph, siga este tutorial: [Uso de la Plataforma de identidad de Microsoft para llamar a Microsoft Graph API desde una aplicación web ASP.NET Core 2.x en nombre de un usuario que inicia sesión con su cuenta profesional y educativa en Microsoft National Cloud](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-4-Sovereign-Call-MSGraph#using-the-microsoft-identity-platform-to-call-the-microsoft-graph-api-from-an-an-aspnet-core-2x-web-app-on-behalf-of-a-user-signing-in-using-their-work-and-school-account-in-microsoft-national-cloud).

## <a name="javascript"></a>[JavaScript](#tab/javascript)

Para habilitar la aplicación MSAL.js para nubes soberanas:

### <a name="step-1-register-your-application"></a>Paso 1: Registrar su aplicación

1. Inicie sesión en [Azure Portal](https://portal.azure.us/).

   Para buscar los puntos de conexión de Azure Portal para otras nubes nacionales, consulte [Puntos de conexión de registro de aplicaciones](authentication-national-cloud.md#app-registration-endpoints).

1. Si la cuenta proporciona acceso a más de un inquilino, seleccione la cuenta en la esquina superior derecha y establezca la sesión del portal en el inquilino de Azure AD deseado.
1. Vaya a la página [Registros de aplicaciones](https://aka.ms/ra/ff) de la plataforma de identidad de Microsoft para desarrolladores.
1. Cuando se abra la página **Registrar una aplicación**, escriba el nombre de su aplicación.
1. En **Tipos de cuenta compatibles**, seleccione **Cuentas en cualquier directorio organizativo**.
1. En la sección **URI de redirección**, seleccione la plataforma **web** y establezca el valor en la dirección URL de la aplicación según el servidor web. Consulte las secciones siguientes para obtener instrucciones sobre cómo establecer y obtener la dirección URL de redireccionamiento en Visual Studio y Node.
1. Seleccione **Registrar**.
1. En la página de **Información general** de la aplicación, anote el valor en **Id. de aplicación (cliente)** .
1. Este tutorial requiere que habilite el [flujo de concesión implícita](v2-oauth2-implicit-grant-flow.md). En el panel izquierdo de la aplicación registrada, seleccione **Autenticación**.
1. En **Configuración avanzada**, en **Concesión implícita**, active las casillas **Tokens de id.** y **Tokens de acceso**. Los tokens de identificador y los tokens de acceso son obligatorios, ya que esta aplicación tiene que iniciar la sesión de los usuarios y llamar a una API.
1. Seleccione **Guardar**.

### <a name="step-2--set-up-your-web-server-or-project"></a>Paso 2:  Configuración de un proyecto o servidor web

- [Descargue los archivos del proyecto](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/quickstart.zip) para un servidor web local, como Node.

  or

- [Descargue el proyecto de Visual Studio](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/vsquickstart.zip).

Luego, vaya a [Configuración de JavaScript SPA](#step-4-configure-your-javascript-spa) para configurar el ejemplo de código antes de ejecutarlo.

### <a name="step-3-use-the-microsoft-authentication-library-to-sign-in-the-user"></a>Paso 3: Uso de la biblioteca de autenticación de Microsoft para iniciar la sesión del usuario

Siga los pasos del [tutorial de JavaScript](tutorial-v2-javascript-spa.md#create-your-project) para crear el proyecto e integrarlo con MSAL e iniciar la sesión del usuario.

### <a name="step-4-configure-your-javascript-spa"></a>Paso 4: Configuración de JavaScript SPA

En el archivo `index.html` creado durante la instalación del proyecto, agregue la información de registro de aplicación. Agregue el código siguiente en la parte superior, dentro de las etiquetas `<script></script>` en el cuerpo del archivo `index.html`:

```javascript
const msalConfig = {
    auth:{
        clientId: "Enter_the_Application_Id_here",
        authority: "https://login.microsoftonline.us/Enter_the_Tenant_Info_Here",
        }
}

const graphConfig = {
        graphEndpoint: "https://graph.microsoft.us",
        graphScopes: ["user.read"],
}

// create UserAgentApplication instance
const myMSALObj = new UserAgentApplication(msalConfig);
```

En el código:

- `Enter_the_Application_Id_here` es el valor del **id. de aplicación (cliente)** para la aplicación que se registró.
- `Enter_the_Tenant_Info_Here` se establece en una de las opciones siguientes:
    - Si la aplicación admite **Solo las cuentas de este directorio organizativo** , reemplace este valor por el identificador de inquilino o el nombre de inquilino (por ejemplo, contoso.microsoft.com).
    - Si la aplicación admite **Cuentas en cualquier directorio organizativo**, reemplace este valor por `organizations`.

    Para buscar los puntos de conexión de autenticación para todas las nubes nacionales, consulte [Puntos de conexión de autenticación de Azure AD](./authentication-national-cloud.md#azure-ad-authentication-endpoints).

    > [!NOTE]
    > No se admiten cuentas de Microsoft personales en las nubes nacionales.

- `graphEndpoint` es el punto de conexión de Microsoft Graph para la nube de Microsoft para el gobierno de Estados Unidos.

   Para buscar los puntos de conexión de Microsoft Graph de todas las nubes nacionales, consulte [Puntos de conexión de Microsoft Graph en nubes nacionales](/graph/deployments#microsoft-graph-and-graph-explorer-service-root-endpoints).

## <a name="python"></a>[Python](#tab/python)

Para habilitar la aplicación MSAL para Python para nubes soberanas:

- Registre la aplicación en un portal específico, en función de la nube. Para más información sobre cómo elegir el portal, consulte [Puntos de conexión de registro de aplicaciones.](authentication-national-cloud.md#app-registration-endpoints)
- Use cualquiera de las [muestras](https://github.com/AzureAD/microsoft-authentication-library-for-python/tree/dev/sample) del repositorio con algunos cambios en la configuración, en función de la nube, que se menciona a continuación.
- Use una autoridad específica, en función de la nube en la que haya registrado la aplicación. Para obtener más información sobre las autoridades de distintas nubes, consulte [Puntos de conexión de Autenticación de Azure AD](authentication-national-cloud.md#azure-ad-authentication-endpoints).

    Este es un ejemplo de autoridad:

    ```json
    "authority": "https://login.microsoftonline.us/Enter_the_Tenant_Info_Here"
    ```

- Para llamar a Microsoft Graph, se requiere una dirección URL de punto de conexión de Graph específica que dependa de la nube que esté usando. Para buscar los puntos de conexión de Microsoft Graph de todas las nubes nacionales, consulte [Puntos de conexión de raíz de servicio de Microsoft Graph y Graph Explorer](/graph/deployments#microsoft-graph-and-graph-explorer-service-root-endpoints).

    A continuación se muestra un ejemplo de un punto de conexión de Graph con el ámbito:

    ```json
    "endpoint" : "https://graph.microsoft.us/v1.0/me"
    "scope": "User.Read"
    ```

## <a name="java"></a>[Java](#tab/java)

Para habilitar la aplicación MSAL para Java para nubes soberanas:

- Registre la aplicación en un portal específico, en función de la nube. Para más información sobre cómo elegir el portal, consulte [Puntos de conexión de registro de aplicaciones.](authentication-national-cloud.md#app-registration-endpoints)
- Use cualquiera de las [muestras](https://github.com/AzureAD/microsoft-authentication-library-for-java/tree/dev/src/samples) del repositorio con algunos cambios en la configuración, en función de la nube, que se menciona a continuación.
- Use una autoridad específica, en función de la nube en la que haya registrado la aplicación. Para obtener más información sobre las autoridades de distintas nubes, consulte [Puntos de conexión de Autenticación de Azure AD](authentication-national-cloud.md#azure-ad-authentication-endpoints).

Este es un ejemplo de autoridad:

```json
"authority": "https://login.microsoftonline.us/Enter_the_Tenant_Info_Here"
```

- Para llamar a Microsoft Graph, se requiere una dirección URL de punto de conexión de Graph específica que dependa de la nube que esté usando. Para buscar los puntos de conexión de Microsoft Graph de todas las nubes nacionales, consulte [Puntos de conexión de raíz de servicio de Microsoft Graph y Graph Explorer](/graph/deployments#microsoft-graph-and-graph-explorer-service-root-endpoints).

A continuación se muestra un ejemplo de un punto de conexión de Graph con el ámbito:

```json
"endpoint" : "https://graph.microsoft.us/v1.0/me"
"scope": "User.Read"
```

## <a name="objective-c"></a>[Objective-C](#tab/objc)

MSAL para iOS y macOS se puede usar para adquirir tokens en nubes nacionales, pero se requiere una configuración adicional al crear `MSALPublicClientApplication`.

Por ejemplo, si quiere que la aplicación sea una aplicación de varios inquilinos en una nube nacional (en la Administración Pública de Estados Unidos), podría escribir lo siguiente:

```objc
MSALAADAuthority *aadAuthority =
                [[MSALAADAuthority alloc] initWithCloudInstance:MSALAzureUsGovernmentCloudInstance
                                                   audienceType:MSALAzureADMultipleOrgsAudience
                                                      rawTenant:nil
                                                          error:nil];

MSALPublicClientApplicationConfig *config =
                [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<your-client-id-here>"
                                                                redirectUri:@"<your-redirect-uri-here>"
                                                                  authority:aadAuthority];

NSError *applicationError = nil;
MSALPublicClientApplication *application =
                [[MSALPublicClientApplication alloc] initWithConfiguration:config error:&applicationError];
```

## <a name="swift"></a>[Swift](#tab/swift)

MSAL para iOS y macOS se puede usar para adquirir tokens en nubes nacionales, pero se requiere una configuración adicional al crear `MSALPublicClientApplication`.

Por ejemplo, si quiere que la aplicación sea una aplicación de varios inquilinos en una nube nacional (en la Administración Pública de Estados Unidos), podría escribir lo siguiente:

```swift
let authority = try? MSALAADAuthority(cloudInstance: .usGovernmentCloudInstance, audienceType: .azureADMultipleOrgsAudience, rawTenant: nil)

let config = MSALPublicClientApplicationConfig(clientId: "<your-client-id-here>", redirectUri: "<your-redirect-uri-here>", authority: authority)
if let application = try? MSALPublicClientApplication(configuration: config) { /* Use application */}
```

---

## <a name="next-steps"></a>Pasos siguientes

Más información sobre:

- [Autenticación en nubes nacionales](authentication-national-cloud.md)
- [Azure Government](../../azure-government/index.yml)
- [Azure China 21Vianet](/azure/china/)
- [Azure Alemania](../../germany/index.yml)