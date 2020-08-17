---
title: Cambios realizados en un proyecto de MVC al conectarse a Azure AD
description: Describe lo que sucede a su proyecto de MVC cuando se conecta a Azure AD mediante los servicios conectados de Visual Studio.
author: ghogen
manager: jillfra
ms.prod: visual-studio-windows
ms.technology: vs-azure
ms.workload: azure-vs
ms.topic: how-to
ms.date: 03/12/2018
ms.author: ghogen
ms.custom: devx-track-csharp, aaddev, vs-azure
ms.openlocfilehash: ab72512b2e5d915dbd4b8a3d7d8446a5932eef9e
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88165454"
---
# <a name="what-happened-to-my-mvc-project-visual-studio-azure-active-directory-connected-service"></a>¿Qué ha ocurrido a mi proyecto MVC (servicio conectado a Azure Active Directory de Visual Studio)?

> [!div class="op_single_selector"]
> - [Introducción](vs-active-directory-dotnet-getting-started.md)
> - [¿Qué ha ocurrido?](vs-active-directory-dotnet-what-happened.md)

En este artículo se identifican los cambios exactos realizados en el proyecto ASP.NET MVC cuando se agrega el [servicio conectado de Azure Active Directory mediante Visual Studio](vs-active-directory-add-connected-service.md).

Para más información sobre cómo trabajar con el servicio conectado, consulte la [introducción](vs-active-directory-dotnet-getting-started.md).

## <a name="added-references"></a>Referencias agregadas

Afecta al archivo del proyecto (referencias *.NET ) y a `packages.config` (referencias de NuGet).

| Tipo | Referencia |
| --- | --- |
| .NET; NuGet | Microsoft.IdentityModel.Protocol.Extensions |
| .NET; NuGet | Microsoft.Owin |
| .NET; NuGet | Microsoft.Owin.Host.SystemWeb |
| .NET; NuGet | Microsoft.Owin.Security |
| .NET; NuGet | Microsoft.Owin.Security.Cookies |
| .NET; NuGet | Microsoft.Owin.Security.OpenIdConnect |
| .NET; NuGet | Owin |
| .NET        | System.IdentityModel |
| .NET; NuGet | System.IdentityModel.Tokens.Jwt |
| .NET        | System.Runtime.Serialization |

Referencias adicionales si ha seleccionado la opción **Leer datos de directorio**:

| Tipo | Referencia |
| --- | --- |
| .NET; NuGet | EntityFramework |
| .NET        | EntityFramework.SqlServer (solo Visual Studio 2015) |
| .NET; NuGet | Microsoft.Azure.ActiveDirectory.GraphClient |
| .NET; NuGet | Microsoft.Data.Edm |
| .NET; NuGet | Microsoft.Data.OData |
| .NET; NuGet | Microsoft.Data.Services.Client |
| .NET; NuGet | Microsoft.IdentityModel.Clients.ActiveDirectory |
| .NET        | Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms (Visual Studio 2015 solo) |
| .NET; NuGet | System.Spatial |

Se quitan las referencias siguientes (solo proyectos de ASP.NET 4, como en Visual Studio 2015):

| Tipo | Referencia |
| --- | --- |
| .NET; NuGet | Microsoft.AspNet.Identity.Core |
| .NET; NuGet | Microsoft.AspNet.Identity.EntityFramework |
| .NET; NuGet | Microsoft.AspNet.Identity.Owin |

## <a name="project-file-changes"></a>Cambios del archivo de proyecto

- Establezca la propiedad `IISExpressSSLPort` en un número distinto.
- Establezca la propiedad `WebProject_DirectoryAccessLevelKey` en 0 o 1 si ha seleccionado la opción **Leer datos de directorio**.
- Establezca la propiedad `IISUrl` en `https://localhost:<port>/`, donde `<port>` coincide con el valor `IISExpressSSLPort`.

## <a name="webconfig-or-appconfig-changes"></a>Cambios de web.config o app.config

- Se agregaron las siguientes entradas de configuración:

    ```xml
    <appSettings>
        <add key="ida:ClientId" value="<ClientId from the new Azure AD app>" />
        <add key="ida:AADInstance" value="https://login.microsoftonline.com/" />
        <add key="ida:Domain" value="<your selected Azure domain>" />
        <add key="ida:TenantId" value="<the Id of your selected Azure AD tenant>" />
        <add key="ida:PostLogoutRedirectUri" value="<project start page, such as https://localhost:44335>" />
    </appSettings>
    ```

- Se agregaron los elementos `<dependentAssembly>` en el nodo `<runtime><assemblyBinding>` para `System.IdentityModel.Tokens.Jwt` y `Microsoft.IdentityModel.Protocol.Extensions`.

Cambios adicionales si seleccionó la opción **Leer datos de directorio**:

- Se agregaron las siguientes entradas de configuración en `<appSettings>`:

    ```xml
    <add key="ida:ClientSecret" value="<Azure AD app's new client secret>" />
    ```

- Se agregaron los siguientes elementos en `<configuration>`; los valores para el archivo project-mdf-file y project-catalog-id variarán:

    ```xml
    <configSections>
      <!-- For more information on Entity Framework configuration, visit https://go.microsoft.com/fwlink/?LinkID=237468 -->
      <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
    </configSections>

    <connectionStrings>
      <add name="DefaultConnection" connectionString="Data Source=(localdb)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\<project-mdf-file>.mdf;Initial Catalog=<project-catalog-id>;Integrated Security=True" providerName="System.Data.SqlClient" />
    </connectionStrings>

    <entityFramework>
      <defaultConnectionFactory type="System.Data.Entity.Infrastructure.LocalDbConnectionFactory, EntityFramework">
        <parameters>
          <parameter value="mssqllocaldb" />
        </parameters>
      </defaultConnectionFactory>
      <providers>
        <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
      </providers>
    </entityFramework>
    ```

- Se agregaron elementos `<dependentAssembly>` en el nodo `<runtime><assemblyBinding>` para `Microsoft.Data.Services.Client`, `Microsoft.Data.Edm` y `Microsoft.Data.OData`.

## <a name="code-changes-and-additions"></a>Adiciones y cambios en el código

- Se agregó el atributo `[Authorize]` a `Controllers/HomeController.cs` y cualquier otro controlador existente.

- Se agregó una clase de inicio de autenticación, `App_Start/Startup.Auth.cs`, que contiene la lógica de inicio para la autenticación de Azure AD. Si seleccionó la opción **Leer datos de directorio**, este archivo también contiene código para recibir un código de OAuth y cambiarlo por un token de acceso.

- Se agregó una clase de controlador, `Controllers/AccountController.cs`, que contiene los métodos `SignIn` y `SignOut`.

- Se agregó una vista parcial, `Views/Shared/_LoginPartial.cshtml`, que contiene un vínculo de acción para `SignIn` y `SignOut`.

- Se agregó una vista parcial, `Views/Account/SignoutCallback.cshtml`, que contiene código HTML para la interfaz de usuario de cierre de sesión.

- Se actualizó el método `Startup.Configuration` para incluir una llamada a `ConfigureAuth(app)` si ya existía la clase; en caso contrario, se agregó una clase `Startup` que incluye llamadas al método.

- Se agregó `Connected Services/AzureAD/ConnectedService.json` (Visual Studio 2017) o `Service References/Azure AD/ConnectedService.json` (Visual Studio 2015), que contiene información que usa Visual Studio para realizar el seguimiento de la adición del servicio conectado.

- Si seleccionó la opción **Leer datos de directorio**, se agregó `Models/ADALTokenCache.cs` y `Models/ApplicationDbContext.cs` para admitir el almacenamiento en caché de tokens. Se agregó también un controlador y una vista adicionales para ilustrar el acceso a la información de perfil de usuario mediante Graph API de Azure: `Controllers/UserProfileController.cs`, `Views/UserProfile/Index.cshtml` y `Views/UserProfile/Relogin.cshtml`

### <a name="file-backup-visual-studio-2015"></a>Copia de seguridad de archivos (Visual Studio 2015)

Al agregar el servicio conectado, Visual Studio 2015 hace una copia de seguridad de los archivos modificados y eliminados. Todos los archivos afectados se guardan en la carpeta `Backup/AzureAD`. En Visual Studio 2017 y versiones posteriores no se crean copias de seguridad.

- `Startup.cs`
- `App_Start\IdentityConfig.cs`
- `App_Start\Startup.Auth.cs`
- `Controllers\AccountController.cs`
- `Controllers\ManageController.cs`
- `Models\IdentityModels.cs`
- `Models\ManageViewModels.cs`
- `Views\Shared\_LoginPartial.cshtml`

## <a name="changes-on-azure"></a>Cambios en Azure

- Se creó una aplicación de Azure AD en el dominio que seleccionó al agregar el servicio conectado.
- Se actualizó la aplicación para incluir el permiso **Leer datos de directorio** si se seleccionó esta opción.

[Más información acerca de Azure Active Directory](https://azure.microsoft.com/services/active-directory/).

## <a name="next-steps"></a>Pasos siguientes

- [Escenarios de autenticación para Azure Active Directory](./authentication-vs-authorization.md)
- [Adición de inicio de sesión con Microsoft a una aplicación web ASP.NET](quickstart-v2-aspnet-webapp.md)