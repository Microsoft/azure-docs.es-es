---
title: Administración de usuarios con Microsoft Graph API
titleSuffix: Azure AD B2C
description: Procedimiento para administrar los usuarios de un inquilino de Azure AD B2C llamando a Microsoft Graph API y mediante una identidad de aplicación para automatizar el proceso.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/03/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 8132eb72b3e448d7ae830b29ccb7dc51528c1250
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87921408"
---
# <a name="manage-azure-ad-b2c-user-accounts-with-microsoft-graph"></a>Administrar cuentas de usuario de Azure AD B2C con Microsoft Graph

Microsoft Graph permite administrar cuentas de usuario en el directorio de Azure AD B2C, ya que proporciona métodos de creación, lectura, actualización y eliminación en Microsoft Graph API. Puede migrar un almacén de usuarios existente a un inquilino de Azure AD B2C y realizar otras operaciones de administración de cuentas de usuario llamando a Microsoft Graph API.

En las secciones siguientes, se presentan los aspectos clave de la administración de usuarios de Azure AD B2C con Microsoft Graph API. Las operaciones, los tipos y las propiedades de Microsoft Graph API que se presentan aquí son un subconjunto de las que aparecen en la documentación de referencia de Microsoft Graph API.

## <a name="register-a-management-application"></a>Registrar una aplicación de administración

Antes de que cualquier aplicación de administración de usuarios o script que escriba pueda interactuar con los recursos de su inquilino de Azure AD B2C, necesita un registro de aplicación que conceda permisos para hacerlo.

Siga los pasos de este artículo de procedimientos para crear un registro de aplicación que pueda usar la aplicación de administración:

[Administrar Azure AD B2C con Microsoft Graph](microsoft-graph-get-started.md)

## <a name="user-management-microsoft-graph-operations"></a>Operaciones de Microsoft Graph para la administración de usuarios

Las siguientes operaciones de administración de usuarios están disponibles en [Microsoft Graph API](https://docs.microsoft.com/graph/api/resources/user):

- [Obtención de una lista de usuarios](https://docs.microsoft.com/graph/api/user-list)
- [Creación de un usuario](https://docs.microsoft.com/graph/api/user-post-users)
- [Obtención de un usuario](https://docs.microsoft.com/graph/api/user-get)
- [Actualizar usuario](https://docs.microsoft.com/graph/api/user-update)
- [Eliminar un usuario](https://docs.microsoft.com/graph/api/user-delete)

## <a name="user-properties"></a>Propiedades de usuario

### <a name="display-name-property"></a>Propiedad de nombre para mostrar

`displayName` es el nombre que se va a mostrar en la administración de usuarios de Azure Portal para el usuario y, en el token de acceso que Azure AD B2C devuelve a la aplicación. Esta propiedad es obligatoria.

### <a name="identities-property"></a>Propiedad de identidades

Una cuenta de cliente, que podría ser un consumidor, un asociado o un ciudadano, puede asociarse con estos tipos de identidad:

- Identidad **Local**: el nombre de usuario y la contraseña almacenados localmente en el directorio de Azure AD B2C. A menudo se hace referencia a estas identidades como "cuentas locales".
- Identidad **Federado**: también denominada cuenta *social* o de *empresa*, la identidad del usuario se administra mediante un proveedor de identidades federado, como Facebook, Microsoft, ADFS o Salesforce.

Un usuario con una cuenta de cliente puede iniciar sesión con varias identidades. Por ejemplo, el nombre de usuario, el correo electrónico, el Id. de empleado, el Id. oficial y otros. Una sola cuenta puede tener varias identidades, tanto locales como sociales, con la misma contraseña.

En Microsoft Graph API, las identidades locales y federadas se almacenan en el atributo `identities` de usuario, que es de tipo [objectIdentity][graph-objectIdentity]. La colección `identities` representa un conjunto de identidades que se usan para iniciar sesión en una cuenta de usuario. Esta colección permite al usuario iniciar sesión en la cuenta de usuario con cualquiera de sus identidades asociadas.

| Propiedad   | Tipo |Descripción|
|:---------------|:--------|:----------|
|signInType|string| Especifica los tipos de inicio de sesión de usuario del directorio. Para la cuenta local: `emailAddress`, `emailAddress1`, `emailAddress2`, `emailAddress3`, `userName` o cualquier otro tipo que desee. La cuenta social debe estar establecida en `federated`.|
|issuer|string|Especifica el emisor de la identidad. En el caso de las cuentas locales (donde **signInType** no es `federated`), esta propiedad es el nombre de dominio predeterminado del inquilino de B2C local, por ejemplo `contoso.onmicrosoft.com`. En el caso de la identidad social (donde **signInType** es `federated`), el valor es el nombre del emisor; por ejemplo, `facebook.com`|
|issuerAssignedId|string|Especifica el identificador único asignado al usuario por el emisor. La combinación de **issuer** e **issuerAssignedId** debe ser única dentro del inquilino. En el caso de la cuenta local, cuando **signInType** se establece en `emailAddress` o `userName`, representa el nombre de inicio de sesión del usuario.<br>Cuando **signInType** se establece en: <ul><li>`emailAddress` (o empieza por `emailAddress` como `emailAddress1`) **issuerAssignedId** debe ser una dirección de correo electrónico válida.</li><li>`userName` (o cualquier otro valor), **issuerAssignedId** debe ser una [parte local válida de una dirección de correo electrónico](https://tools.ietf.org/html/rfc3696#section-3).</li><li>`federated`, **issuerAssignedId** representa el identificador único de la cuenta federada.</li></ul>|

La siguiente propiedad **Identities**, con una identidad de cuenta local con un nombre de inicio de sesión, una dirección de correo electrónico como inicio de sesión y con una identidad social. 

 ```json
 "identities": [
     {
       "signInType": "userName",
       "issuer": "contoso.onmicrosoft.com",
       "issuerAssignedId": "johnsmith"
     },
     {
       "signInType": "emailAddress",
       "issuer": "contoso.onmicrosoft.com",
       "issuerAssignedId": "jsmith@yahoo.com"
     },
     {
       "signInType": "federated",
       "issuer": "facebook.com",
       "issuerAssignedId": "5eecb0cd"
     }
   ]
 ```

En las identidades federadas, en función del proveedor de identidades, **issuerAssignedId** es un valor único para un usuario determinado por aplicación o cuenta de desarrollo. Configure la directiva de Azure AD B2C con el mismo identificador de aplicación que asignó previamente el proveedor de redes sociales u otra aplicación con la misma cuenta de desarrollo.

### <a name="password-profile-property"></a>Propiedad de perfil de contraseña

En el caso de una identidad local, se requiere la propiedad **passwordProfile** y contiene la contraseña del usuario. La propiedad `forceChangePasswordNextSignIn` debe establecerse en `false`.

En el caso de una identidad federada (social), no se requiere la propiedad **passwordProfile**.

```json
"passwordProfile" : {
    "password": "password-value",
    "forceChangePasswordNextSignIn": false
  }
```

### <a name="password-policy-property"></a>Propiedad de directiva de contraseñas

La directiva de contraseñas de Azure AD B2C (para cuentas locales) se basa en la directiva de [seguridad de contraseña segura](../active-directory/authentication/concept-sspr-policy.md) de Azure Active Directory. Las directivas de restablecimiento de la contraseña, registro e inicio de sesión de Azure AD B2C requieren esta seguridad de contraseña segura y las contraseñas no expiran.

En los escenarios de migración de usuarios, si las cuentas que se van a migrar cuentan con una seguridad de contraseña inferior a la [seguridad de contraseña segura](../active-directory/authentication/concept-sspr-policy.md) que exige Azure AD B2C, puede deshabilitar el requisito de contraseña segura. Para cambiar la directiva de contraseñas predeterminada, establezca la propiedad `passwordPolicies` en `DisableStrongPassword`. Por ejemplo, puede modificar la solicitud de creación de usuario de la siguiente manera:

```json
"passwordPolicies": "DisablePasswordExpiration, DisableStrongPassword"
```

### <a name="extension-properties"></a>Propiedades de extensión

Todas las aplicaciones orientadas al cliente tienen requisitos únicos para la información que debe recopilarse. El inquilino de Azure AD B2C incluye un conjunto integrado de información almacenada en propiedades, como el nombre propio, los apellidos, la ciudad y el código postal. Con Azure AD B2C, puede ampliar el conjunto de propiedades que se almacenan en cada cuenta de cliente. Para obtener más información sobre la definición de atributos personalizados, consulte los [atributos personalizados (flujos de usuario)](user-flow-custom-attributes.md) y los [atributos personalizados (directivas personalizadas)](custom-policy-custom-attributes.md).

Microsoft Graph API admite la creación y actualización de un usuario con atributos de extensión. Los atributos de extensión de Graph API se denominan mediante la convención `extension_ApplicationClientID_attributename`, donde `ApplicationClientID` es el **id. de la aplicación (cliente)** de la aplicación `b2c-extensions-app` (se encuentra en la opción **Registros de aplicaciones** > **Todas las aplicaciones** en Azure Portal). Tenga en cuenta que el **id. de la aplicación (cliente)** , tal como se representa en el nombre del atributo de extensión, no incluye guiones. Por ejemplo:

```json
"extension_831374b3bd5041bfaa54263ec9e050fc_loyaltyNumber": "212342"
```

## <a name="code-sample-how-to-programmatically-manage-user-accounts"></a>Ejemplo de código: Administración de cuentas de usuario mediante programación

Este ejemplo de código es una aplicación de consola .NET Core que usa el [SDK de Microsoft Graph](https://docs.microsoft.com/graph/sdks/sdks-overview) para interactuar con Microsoft Graph API. En su código se muestra cómo llamar a la API para administrar usuarios mediante programación en un inquilino de Azure AD B2C.
Puede [descargar el archivo de ejemplo](https://github.com/Azure-Samples/ms-identity-dotnetcore-b2c-account-management/archive/master.zip) (*.zip), [examinar el repositorio](https://github.com/Azure-Samples/ms-identity-dotnetcore-b2c-account-management) de GitHub o clonar el repositorio:

```cmd
git clone https://github.com/Azure-Samples/ms-identity-dotnetcore-b2c-account-management.git
```

Después de obtener el ejemplo de código, configúrelo para su entorno y, a continuación, compile el proyecto:

1. Abra el proyecto en [Visual Studio](https://visualstudio.microsoft.com) o en [Visual Studio Code](https://code.visualstudio.com).
1. Abra `src/appsettings.json`.
1. En la sección `appSettings`, reemplace `your-b2c-tenant` por el nombre de su inquilino, y `Application (client) ID` y `Client secret` por los valores para el registro de la aplicación de administración (consulte la sección de este artículo [Registrar una aplicación de administración](#register-a-management-application)).
1. Abra una ventana de consola en el clon local del repositorio, cambie al directorio `src` y, a continuación, compile el proyecto:
    ```console
    cd src
    dotnet build
    ```
1. Ejecute la aplicación con el comando `dotnet`:

    ```console
    dotnet bin/Debug/netcoreapp3.0/b2c-ms-graph.dll
    ```

La aplicación muestra una lista de los comandos que se pueden ejecutar. Por ejemplo, obtener todos los usuarios, obtener un solo usuario, eliminar un usuario, actualizar la contraseña de un usuario y realizar una importación en bloque.

### <a name="code-discussion"></a>Discusión de código

En el código de ejemplo se usa el [SDK de Microsoft Graph](https://docs.microsoft.com/graph/sdks/sdks-overview), que se ha diseñado para simplificar la creación de aplicaciones de alta calidad, eficientes y resistentes que tienen acceso a Microsoft Graph.

Cualquier solicitud a Microsoft Graph API requiere un token de acceso para la autenticación. La solución usa el paquete de NuGet [Microsoft.Graph.Auth](https://www.nuget.org/packages/Microsoft.Graph.Auth/) que proporciona un contenedor basado en escenario de autenticación de la biblioteca de autenticación de Microsoft (MSAL) para su uso con el SDK de Microsoft Graph.

El método `RunAsync` en el archivo _Program.cs_:

1. Lee la configuración de la aplicación del archivo _appsettings.json_.
1. Inicializa el proveedor de autenticación mediante el flujo de [concesión de credenciales de cliente de OAuth 2.0](../active-directory/develop/v2-oauth2-client-creds-grant-flow.md). Con el flujo de concesión de credenciales de cliente, la aplicación puede obtener un token de acceso para llamar a Microsoft Graph API.
1. Configura el cliente del servicio de Microsoft Graph con el proveedor de autenticación:

    ```csharp
    // Read application settings from appsettings.json (tenant ID, app ID, client secret, etc.)
    AppSettings config = AppSettingsFile.ReadFromJsonFile();

    // Initialize the client credential auth provider
    IConfidentialClientApplication confidentialClientApplication = ConfidentialClientApplicationBuilder
        .Create(config.AppId)
        .WithTenantId(config.TenantId)
        .WithClientSecret(config.ClientSecret)
        .Build();
    ClientCredentialProvider authProvider = new ClientCredentialProvider(confidentialClientApplication);

    // Set up the Microsoft Graph service client with client credentials
    GraphServiceClient graphClient = new GraphServiceClient(authProvider);
    ```

A continuación, se usa la instancia inicializada de *GraphServiceClient* en _GraphService.cs_ para realizar las operaciones de administración de usuarios. Por ejemplo, obtener una lista de las cuentas de usuario en el inquilino:

```csharp
public static async Task ListUsers(GraphServiceClient graphClient)
{
    Console.WriteLine("Getting list of users...");

    // Get all users (one page)
    var result = await graphClient.Users
        .Request()
        .Select(e => new
        {
            e.DisplayName,
            e.Id,
            e.Identities
        })
        .GetAsync();

    foreach (var user in result.CurrentPage)
    {
        Console.WriteLine(JsonConvert.SerializeObject(user));
    }
}
```

El documento [Realizar llamadas API con los SDK de Microsoft Graph](https://docs.microsoft.com/graph/sdks/create-requests) incluye información sobre cómo leer y escribir información de Microsoft Graph, usar `$select` para controlar las propiedades devueltas, proporcionar parámetros de consulta personalizados y usar los parámetros de consulta `$filter` y `$orderBy`.

## <a name="next-steps"></a>Pasos siguientes

Para obtener un índice completo de las operaciones de Microsoft Graph API compatibles con los recursos de Azure AD B2C, vea [Operaciones de Microsoft Graph disponibles para Azure AD B2C](microsoft-graph-operations.md).

<!-- LINK -->

[graph-objectIdentity]: https://docs.microsoft.com/graph/api/resources/objectidentity
[graph-user]: (https://docs.microsoft.com/graph/api/resources/user)
