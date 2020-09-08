---
title: Adquisición de un token de Azure AD para la autorización de solicitudes desde una aplicación cliente
titleSuffix: Azure Storage
description: Use Azure Active Directory para autenticarse desde una aplicación cliente, adquiera un token de OAuth 2.0 y autorice solicitudes de Azure Blob Storage y Queue Storage.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 06/22/2020
ms.author: tamram
ms.subservice: common
ms.custom: has-adal-ref, devx-track-csharp
ms.openlocfilehash: d842974b0b53e0b0ce199334a07f11e5c998b18d
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018814"
---
# <a name="acquire-a-token-from-azure-ad-for-authorizing-requests-from-a-client-application"></a>Adquisición de un token de Azure AD para la autorización de solicitudes desde una aplicación cliente

Una ventaja clave del uso de Azure Active Directory (Azure AD) con Azure Blob Storage o Queue Storage es que ya no necesita almacenar las credenciales en el código. En su lugar, puede solicitar un token de acceso de OAuth 2.0 desde la Plataforma de identidad de Microsoft (anteriormente Azure AD). Azure AD autentica la entidad de seguridad (un usuario, grupo o entidad de servicio) ejecutando la aplicación. Si la autenticación se realiza correctamente, Azure AD devuelve el token de acceso a la aplicación y la aplicación puede entonces usar el token de acceso para autorizar las solicitudes de Azure Blob Storage o Queue Storage.

En este artículo se le muestra cómo configurar su aplicación web o aplicación nativa para la autenticación con la Plataforma de identidad de Microsoft 2.0. El ejemplo de código incluye .NET, pero otros lenguajes utilizan un enfoque similar. Para obtener más información sobre la Plataforma de identidad de Microsoft 2.0, consulte la [Introducción a la Plataforma de identidad de Microsoft (v2.0)](../../active-directory/develop/v2-overview.md).

Para información general sobre el flujo de concesión de código OAuth 2.0, consulte [Autorización del acceso a aplicaciones web de Azure Active Directory mediante el flujo de concesión de código OAuth 2.0](../../active-directory/develop/v2-oauth2-auth-code-flow.md).

## <a name="assign-a-role-to-an-azure-ad-security-principal"></a>Asignación de un rol a una entidad de seguridad de Azure AD

Para autenticar a una entidad de seguridad de la aplicación de Azure Storage, primero configure los valores del control de acceso basado en roles (RBAC) para esa entidad de seguridad. Azure Storage define los roles integrados de Azure que abarcan los permisos para los contenedores y las colas. Cuando el rol de Azure se asigna a una entidad de seguridad, esta recibe acceso a ese recurso. Para obtener más información, consulte el artículo sobre cómo [administrar los derechos de acceso a los datos de Azure Blob y Queue con RBAC](storage-auth-aad-rbac.md).

## <a name="register-your-application-with-an-azure-ad-tenant"></a>Registro de la aplicación con un inquilino de Azure AD

El primer paso para usar Azure AD con el fin de autorizar el acceso a los recursos de almacenamiento es registrar la aplicación cliente con un inquilino desde [Azure Portal](https://portal.azure.com). Al registrar la aplicación cliente, facilita información acerca de la aplicación a Azure AD. Azure AD proporciona un identificador de cliente (también denominado *identificador de aplicación*) que se utiliza para asociar la aplicación con Azure AD en runtime. Para más información sobre el identificador de cliente, consulte [Objetos de aplicación y de entidad de servicio de Azure Active Directory](../../active-directory/develop/app-objects-and-service-principals.md).

Para registrar su aplicación de Azure Storage, siga los pasos mostrados en [Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft](../../active-directory/develop/quickstart-configure-app-access-web-apis.md). En la siguiente imagen se muestra la configuración común para registrar una aplicación web:

![Captura de pantalla que muestra cómo registrar la aplicación de almacenamiento con Azure AD](./media/storage-auth-aad-app/app-registration.png)

> [!NOTE]
> Si registra la aplicación como una aplicación nativa, puede especificar cualquier URI válido para el **URI de redirección**. Para las aplicaciones nativas, no es necesario que este valor sea una dirección URL real. Para las aplicaciones web, el URI de redirección debe ser un URI válido, ya que especifica la dirección URL a la que se proporcionan los tokens.

Una vez que ha registrado su aplicación, verá el identificador de la aplicación (o identificador de cliente) en **Configuración**:

![Captura de pantalla que muestra el identificador de cliente](./media/storage-auth-aad-app/app-registration-client-id.png)

Para más información sobre cómo registrar una aplicación con Azure AD, consulte [Integración de aplicaciones con Azure Active Directory](../../active-directory/develop/quickstart-v2-register-an-app.md).

## <a name="grant-your-registered-app-permissions-to-azure-storage"></a>Concesión de permisos a la aplicación registrada para Azure Storage

A continuación, conceda los permisos de aplicación para llamar a las API de Azure Storage. Este paso permite a la aplicación autorizar las solicitudes a Azure Storage con Azure AD.

1. En la página **Información general** de su aplicación registrada, seleccione **Ver permisos de API**.
1. En la sección **Permisos de API**, seleccione **Agregar un permiso** y elija **API de Microsoft**.
1. En la lista de resultados, seleccione **Azure Storage** para mostrar el panel **Solicitud de permisos de API**.
1. En **¿Qué tipo de permiso necesita la aplicación web?** , observe que el tipo de permiso disponible es **Permisos delegados**. Esta opción se selecciona automáticamente de forma predeterminada.
1. En la sección **Selección de permisos** del panel **Solicitud de permisos de API**, active la casilla **user_impersonation** y, a continuación, haga clic en **Agregar permisos**.

    ![Captura de pantalla que muestra los permisos para el almacenamiento](media/storage-auth-aad-app/registered-app-permissions-1.png)

El panel **Permisos de API** muestra ahora que la aplicación de Azure AD registrada tiene acceso a las API de Microsoft Graph y de Azure Storage. La primera vez que registra la aplicación con Azure AD, se conceden permisos a Microsoft Graph automáticamente.

![Captura de pantalla que muestra los permisos de aplicación registrada](media/storage-auth-aad-app/registered-app-permissions-2.png)

## <a name="create-a-client-secret"></a>Creación de un secreto de cliente

La aplicación necesita un secreto de cliente para demostrar su identidad al solicitar un token. Para agregar el secreto de cliente, siga estos pasos:

1. Vaya a su registro de aplicaciones en Azure Portal.
1. Seleccione la configuración de **Certificados y secretos**.
1. En **Secretos de cliente**, haga clic en **Nuevo secreto de cliente** para crear un nuevo secreto.
1. Proporcione una descripción para el secreto y elija el intervalo de expiración deseado.
1. Copie inmediatamente el valor del nuevo secreto en una ubicación segura. El valor completo se le muestra solo una vez.

    ![Captura de pantalla que muestra el secreto de cliente](media/storage-auth-aad-app/client-secret.png)

## <a name="client-libraries-for-token-acquisition"></a>Bibliotecas cliente para la adquisición de tokens

Una vez que haya registrado su aplicación y le haya concedido permisos para obtener acceso a los datos en Azure Blob Storage o Queue Storage, podrá agregar código a su aplicación para autenticar una entidad de seguridad y adquirir un token de OAuth 2.0. Para autenticar y adquirir el token, puede usar una de las [bibliotecas de autenticación de la Plataforma de identidad de Microsoft](../../active-directory/develop/reference-v2-libraries.md) u otra biblioteca de código abierto que admita OpenID Connect 1.0. A continuación, su aplicación puede usar el token de acceso para autorizar una solicitud en Azure Blob Storage o Queue Storage.

Para ver una lista de escenarios en los que se admite la adquisición de tokens, consulte la sección [Flujos de autenticación](/en-us/azure/active-directory/develop/msal-authentication-flows) del [contenido de la biblioteca de autenticación de Microsoft](/azure/active-directory/develop/msal-overview).

## <a name="well-known-values-for-authentication-with-azure-ad"></a>Valores conocidos para la autenticación con Azure AD

Para autenticar una entidad de seguridad con Azure AD, tiene que incluir algunos valores conocidos en el código.

### <a name="azure-ad-authority"></a>Autoridad de Azure AD

Para la nube pública de Microsoft, la autoridad base de Azure AD es la siguiente, donde *tenant-id* es el identificador de inquilino de Active Directory (o identificador de directorio):

`https://login.microsoftonline.com/<tenant-id>/`

El identificador de inquilino identifica el inquilino de Azure AD que se usará para la autenticación. También se conoce como identificador de directorio. Para recuperar el identificador de inquilino, vaya a la página **Información general** para su registro de aplicaciones en Azure Portal y copie el valor desde allí.

### <a name="azure-storage-resource-id"></a>Identificador de recurso de Azure Storage

[!INCLUDE [storage-resource-id-include](../../../includes/storage-resource-id-include.md)]

## <a name="net-code-example-create-a-block-blob"></a>Ejemplo de código .NET: Creación de un blob en bloques

El ejemplo de código muestra cómo obtener un token de acceso de Azure AD. El token de acceso se utiliza para autenticar el usuario especificado y, a continuación, autorizar una solicitud para crear un blob en bloques. Para que este ejemplo funcione, siga primero los pasos descritos en las secciones anteriores.

Para solicitar el token, necesitará los siguientes valores del registro de la aplicación:

- El nombre de su dominio de Azure AD. Recupere este valor de la página **Información general** de su instancia de Azure Active Directory.
- El identificador de inquilino (o directorio). Recupere este valor de la página **Información general** de su registro de aplicaciones.
- El identificador de cliente (o aplicación). Recupere este valor de la página **Información general** de su registro de aplicaciones.
- El URI de redirección de cliente. Recupere este valor de la configuración de **Autenticación** para su registro de aplicaciones.
- El valor del secreto de cliente. Recupere este valor de la ubicación en la que lo copió anteriormente.

### <a name="create-a-storage-account-and-container"></a>Creación de una cuenta de almacenamiento y un contenedor

Para ejecutar el ejemplo de código, cree una cuenta de almacenamiento en la misma suscripción que su instancia de Azure Active Directory. A continuación, cree un contenedor en esa cuenta de almacenamiento. El código de ejemplo creará un blob en bloques en este contenedor.

A continuación, asigne explícitamente el rol **Colaborador de datos de Storage Blob** a la cuenta de usuario en la que ejecutará el código de ejemplo. Para ver instrucciones sobre cómo asignar este rol en Azure Portal, consulte [Conceder acceso a datos de blob y cola de Azure con RBAC en Azure Portal](storage-auth-aad-rbac-portal.md).

> [!NOTE]
> Al crear una cuenta de Azure Storage, no se le asignan automáticamente permisos para tener acceso a datos a través de Azure AD. Tiene que asignarse a sí mismo de forma explícita un rol de Azure para Azure Storage. Puede asignarlo al nivel de su suscripción, grupo de recursos, cuenta de almacenamiento o un contenedor o cola.

### <a name="create-a-web-application-that-authorizes-access-to-blob-storage-with-azure-ad"></a>Creación de una aplicación web que autoriza el acceso a Blob Storage con Azure AD

Cuando la aplicación obtiene acceso a Azure Storage, lo hace en nombre del usuario, lo que significa que se tiene acceso a los recursos de blob o cola con los permisos del usuario que inicia sesión. Para probar este ejemplo de código, necesita una aplicación web que pida al usuario que inicie sesión con una identidad de Azure AD. Puede crear una propia, o bien usar la aplicación de ejemplo proporcionada por Microsoft.

Una aplicación web de ejemplo completada que adquiere un token y lo usa para crear un blob en Azure Storage está disponible en [GitHub](https://aka.ms/aadstorage). La revisión y ejecución del ejemplo completado puede resultar útil para describir los ejemplos de código. Para ver instrucciones sobre cómo ejecutar el ejemplo completado, consulte la sección sobre [visualización y ejecución del ejemplo completado](#view-and-run-the-completed-sample).

#### <a name="add-references-and-using-statements"></a>Incorporación de referencias y uso de instrucciones  

En Visual Studio, instale la biblioteca cliente de Azure Storage. En el menú **Herramientas**, seleccione **Administrador de paquetes NuGet** y después **Consola del Administrador de paquetes**. Escriba los siguientes comandos en la ventana de consola para instalar los paquetes necesarios de la biblioteca cliente de Azure Storage para .NET:

```console
Install-Package Microsoft.Azure.Storage.Blob
Install-Package Microsoft.Azure.Storage.Common
```

A continuación, agregue lo siguiente mediante instrucciones al archivo HomeController.cs:

```csharp
using Microsoft.Identity.Client; //MSAL library for getting the access token
using Microsoft.WindowsAzure.Storage.Auth;
using Microsoft.WindowsAzure.Storage.Blob;
```

#### <a name="create-a-block-blob"></a>Creación de un blob en bloques

Agregue el siguiente fragmento de código para crear un blob en bloques:

```csharp
private static async Task<string> CreateBlob(string accessToken)
{
    // Create a blob on behalf of the user
    TokenCredential tokenCredential = new TokenCredential(accessToken);
    StorageCredentials storageCredentials = new StorageCredentials(tokenCredential);

    // Replace the URL below with your storage account URL
    CloudBlockBlob blob =
        new CloudBlockBlob(
            new Uri("https://<storage-account>.blob.core.windows.net/<container>/Blob1.txt"),
            storageCredentials);
    await blob.UploadTextAsync("Blob created by Azure AD authenticated user.");
    return "Blob successfully created";
}
```

> [!NOTE]
> Para autorizar las operaciones de blob y cola con un token de OAuth 2.0, debe usar HTTPS.

En el ejemplo anterior, la biblioteca cliente de .NET controla la autorización de la solicitud para crear el blob en bloques. Las bibliotecas cliente de Azure Storage también controlan la autorización de la solicitud por usted. Sin embargo, si llama a una operación de Azure Storage con un token de OAuth mediante la API de REST, deberá autorizar la solicitud con el token de OAuth.

Para llamar a las operaciones Blob y Queue service con tokens de acceso de OAuth, pase el token de acceso del encabezado **Autorización** con el esquema **Portador** y especifique una versión de servicio 2017-11-09 o superior, como se muestra en el ejemplo siguiente:

```https
GET /container/file.txt HTTP/1.1
Host: mystorageaccount.blob.core.windows.net
x-ms-version: 2017-11-09
Authorization: Bearer eyJ0eXAiOnJKV1...Xd6j
```

#### <a name="get-an-oauth-token-from-azure-ad"></a>Obtención de un token de OAuth de Azure AD

A continuación, agregue un método que solicite un token a Azure AD en nombre del usuario. Este método define el ámbito para el que se concederán los permisos. Para más información sobre los ámbitos y los permisos, consulte [Permisos y consentimiento en el punto de conexión de la Plataforma de identidad de Microsoft](../../active-directory/develop/v2-permissions-and-consent.md).

Use el identificador de recurso para construir el ámbito de adquisición del token. En el ejemplo se crea el ámbito mediante el identificador de recurso junto con el ámbito `user_impersonation` integrado, que indica que el token se está solicitando en nombre del usuario.

Tenga en cuenta que es posible que deba presentar al usuario una interfaz que le permita dar su consentimiento a la solicitud del token en su nombre. Cuando se necesita consentimiento, el ejemplo detecta **MsalUiRequiredException** y llama a otro método para facilitar la solicitud de consentimiento:

```csharp
public async Task<IActionResult> Blob()
{
    var scopes = new string[] { "https://storage.azure.com/user_impersonation" };
    try
    {
        var accessToken =
            await _tokenAcquisition.GetAccessTokenOnBehalfOfUser(HttpContext, scopes);
        ViewData["Message"] = await CreateBlob(accessToken);
        return View();
    }
    catch (MsalUiRequiredException ex)
    {
        AuthenticationProperties properties =
            BuildAuthenticationPropertiesForIncrementalConsent(scopes, ex);
        return Challenge(properties);
    }
}
```

El consentimiento es el proceso por el cual un usuario concede autorización para que una aplicación acceda a recursos protegidos en su nombre. La Plataforma de identidad de Microsoft 2.0 admite el consentimiento incremental, lo que significa que una entidad de seguridad puede solicitar un conjunto de permisos mínimo de forma inicial y agregar permisos con el tiempo según sea necesario. Si el código solicita un token de acceso, especifique el ámbito de los permisos que necesita la aplicación en un momento dado en el parámetro `scope`. Para obtener más información sobre el consentimiento incremental, consulte la sección titulada **Consentimiento incremental y dinámico** en [¿Por qué actualizar a la Plataforma de identidad de Microsoft (v2.0)?](../../active-directory/azuread-dev/azure-ad-endpoint-comparison.md#incremental-and-dynamic-consent).

El siguiente método crea las propiedades de autenticación para solicitar el consentimiento incremental:

```csharp
private AuthenticationProperties BuildAuthenticationPropertiesForIncrementalConsent(string[] scopes,
                                                                                    MsalUiRequiredException ex)
{
    AuthenticationProperties properties = new AuthenticationProperties();

    // Set the scopes, including the scopes that MSAL.NET needs for the token cache.
    string[] additionalBuildInScopes = new string[] { "openid", "offline_access", "profile" };
    properties.SetParameter<ICollection<string>>(OpenIdConnectParameterNames.Scope,
                                                 scopes.Union(additionalBuildInScopes).ToList());

    // Attempt to set the login_hint so that the logged-in user is not presented
    // with an account selection dialog.
    string loginHint = HttpContext.User.GetLoginHint();
    if (!string.IsNullOrWhiteSpace(loginHint))
    {
        properties.SetParameter<string>(OpenIdConnectParameterNames.LoginHint, loginHint);

        string domainHint = HttpContext.User.GetDomainHint();
        properties.SetParameter<string>(OpenIdConnectParameterNames.DomainHint, domainHint);
    }

    // Specify any additional claims that are required (for instance, MFA).
    if (!string.IsNullOrEmpty(ex.Claims))
    {
        properties.Items.Add("claims", ex.Claims);
    }

    return properties;
}
```

## <a name="view-and-run-the-completed-sample"></a>Visualización y ejecución del ejemplo completado

Para ejecutar la aplicación de ejemplo, antes debe clonarla o descargarla desde [GitHub](https://github.com/Azure-Samples/storage-dotnet-azure-ad-msal). A continuación, actualice la aplicación como se describe en las siguientes secciones.

### <a name="provide-values-in-the-settings-file"></a>Especificación de valores en el archivo de configuración

A continuación, actualice el archivo *appsettings.json* con sus propios valores, de la siguiente manera:

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "<azure-ad-domain-name>.onmicrosoft.com",
    "TenantId": "<tenant-id>",
    "ClientId": "<client-id>",
    "CallbackPath": "/signin-oidc",
    "SignedOutCallbackPath ": "/signout-callback-oidc",

    // To call an API
    "ClientSecret": "<client-secret>"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

### <a name="update-the-storage-account-and-container-name"></a>Actualización del nombre del contenedor y la cuenta de almacenamiento

En el archivo *HomeController.cs*, actualice el URI que hace referencia al blob en bloques para usar el nombre de su cuenta de almacenamiento y contenedor:

```csharp
CloudBlockBlob blob = new CloudBlockBlob(
                      new Uri("https://<storage-account>.blob.core.windows.net/<container>/Blob1.txt"),
                      storageCredentials);
```

### <a name="enable-implicit-grant-flow"></a>Habilitación del flujo de concesión implícita

Para ejecutar el ejemplo, es posible que deba configurar el flujo de concesión implícita para su registro de aplicaciones. Siga estos pasos:

1. Vaya a su registro de aplicaciones en Azure Portal.
1. En la sección **Administrar**, seleccione la configuración de **Autenticación**.
1. En la sección **Concesión implícita**, active la casilla para habilitar los tokens de identificador, como se muestra en la siguiente imagen:

    ![Captura de pantalla que muestra cómo habilitar la configuración para el flujo de concesión implícita](media/storage-auth-aad-app/enable-implicit-grant-flow.png)

### <a name="update-the-port-used-by-localhost"></a>Actualización del puerto usado por localhost

Al ejecutar el ejemplo, es posible que tenga que actualizar el URI de redirección especificado en su registro de aplicaciones para usar el puerto *localhost* asignado en tiempo de ejecución. Para actualizar el URI de redirección a fin de usar el puerto asignado, siga estos pasos:

1. Vaya a su registro de aplicaciones en Azure Portal.
1. En la sección **Administrar**, seleccione la configuración de **Autenticación**.
1. En **URI de redirección**, edite el puerto para que coincida con el usado por la aplicación de ejemplo, como se muestra en la siguiente imagen:

    ![Captura de pantalla que muestra los URI de redirección para el registro de aplicaciones](media/storage-auth-aad-app/redirect-uri.png)

## <a name="next-steps"></a>Pasos siguientes

- [Plataforma de identidad de Microsoft](https://docs.microsoft.com/azure/active-directory/develop/)
- [Administración de los derechos de acceso a los datos de almacenamiento con RBAC](storage-auth-aad-rbac.md)
- [Autenticación del acceso a blobs y colas con Azure Active Directory e identidades administradas para los recursos de Azure](storage-auth-aad-msi.md)
