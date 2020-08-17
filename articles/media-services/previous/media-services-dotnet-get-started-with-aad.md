---
title: Uso de la autenticación de Azure AD para acceder a la API de Azure Media Services con .NET | Microsoft Docs
description: En este tema se explica cómo usar la autenticación de Azure Active Directory (Azure AD) para acceder a la API de Azure Media Services (AMS) con .NET
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: has-adal-ref
ms.openlocfilehash: c1bf77881f672bf3bd2b2dccf91c30855c520052
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87042906"
---
# <a name="use-azure-ad-authentication-to-access-azure-media-services-api-with-net"></a>Uso de la autenticación de Azure AD para acceder a la API de Azure Media Services con .NET

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

A partir de windowsazure.mediaservices 4.0.0.4, Azure Media Services admite la autenticación basada en Azure Active Directory (Azure AD). En este tema se explica cómo usar la autenticación de Azure AD para acceder a la API de Azure Media Services con Microsoft .NET.

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure. Para más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/).
- Una cuenta de Media Services. Para más información, vea [Creación de una cuenta de Azure Media Services mediante Azure Portal](media-services-portal-create-account.md).
- El último paquete [NuGet](https://www.nuget.org/packages/windowsazure.mediaservices).
- Familiarícese con el tema de [información general sobre el acceso a la API de Azure Media Services con la autenticación de Azure AD](media-services-use-aad-auth-to-access-ams-api.md).

Si usa la autenticación de Azure AD con Azure Media Services, puede autenticarse de alguna de estas dos formas:

- **Autenticación de usuario** autentica a una persona que está usando la aplicación para interactuar con los recursos de Azure Media Services. La aplicación interactiva en primer lugar debe solicitar al usuario las credenciales. Un ejemplo es una aplicación de consola de administración que usan los usuarios autorizados para supervisar trabajos de codificación o streaming en vivo.
- **Autenticación de entidad de servicio** autentica un servicio. Las aplicaciones que normalmente utilizan este método de autenticación son aplicaciones que ejecutan servicios de demonio, servicios de nivel intermedio o trabajos programados, como Web Apps, Function Apps, Logic Apps, API o microservicios.

>[!IMPORTANT]
>Azure Media Services actualmente admite un modelo de autenticación de Azure Access Control Service. Pero la autorización de Access Control dejará de usarse el 22 de junio de 2018. Se recomienda migrar tan pronto como sea posible a un modelo de autenticación de Azure Active Directory.

## <a name="get-an-azure-ad-access-token"></a>Obtención de un token de acceso de Azure AD

Para conectarse a la API de Azure Media Services con la autenticación de Azure AD, la aplicación cliente debe solicitar un token de acceso de Azure AD. Si usa el SDK del cliente .NET de Media Services .NET, muchos de los detalles sobre cómo adquirir un token de acceso de Azure AD se ajustan y simplifican en función del caso en las clases [AzureAdTokenProvider](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.Authentication/AzureAdTokenProvider.cs) y [AzureAdTokenCredentials](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.Authentication/AzureAdTokenCredentials.cs).

Por ejemplo, no es necesario proporcionar el URI de autoridad de Azure AD, el URI de recurso de Media Services ni los detalles de la aplicación nativa de Azure AD. Se trata de valores conocidos que ya están configurados por la clase de proveedor de tokens de acceso de Azure AD.

Si no utiliza el SDK de Azure Media Services para .NET, se recomienda que use la [biblioteca de autenticación de Azure AD](../../active-directory/azuread-dev/active-directory-authentication-libraries.md). Para obtener los valores de los parámetros necesarios para usarlos con la biblioteca de autenticación de Azure AD, vea [Uso de Azure Portal para acceder a la configuración de autenticación de Azure AD](media-services-portal-get-started-with-aad.md).

También tiene la opción de sustituir la implementación predeterminada de **AzureAdTokenProvider** por su propia implementación.

## <a name="install-and-configure-azure-media-services-net-sdk"></a>Instalación y configuración del SDK de Azure Media Services para .NET

>[!NOTE]
>Para usar la autenticación de Azure AD con el SDK de Media Services para .NET, debe disponer del último paquete [NuGet](https://www.nuget.org/packages/windowsazure.mediaservices). Además, agregue una referencia al ensamblado **Microsoft.IdentityModel.Clients.ActiveDirectory**. Si usa una aplicación existente, incluya el ensamblado **Microsoft.WindowsAzure.MediaServices.Client.Common.Authentication.dll**.

1. Cree una aplicación de consola en C# mediante Visual Studio.
2. Use el paquete NuGet [windowsazure.mediaservices](https://www.nuget.org/packages/windowsazure.mediaservices) para instalar el **SDK de Azure Media Services para .NET**.

    Para agregar referencias con NuGet, siga estos pasos: en el **Explorador de soluciones**, haga clic con el botón derecho en el nombre del proyecto y, después, seleccione **Administrar paquetes NuGet**. Después, busque **windowsazure.mediaservices** y seleccione **Instalar**.

    O bien

    Ejecute el siguiente comando en la **Consola del Administrador de paquetes** de Visual Studio.

    ```console
    Install-Package windowsazure.mediaservices -Version 4.0.0.4
    ```

3. Agregue **using** al código fuente.

    ```csharp
    using Microsoft.WindowsAzure.MediaServices.Client;
    ```

## <a name="use-user-authentication"></a>Uso de la autenticación de usuario

Para conectarse a la API de Azure Media Services mediante la opción de autenticación de usuario, la aplicación cliente debe solicitar un token de Azure AD con el uso de los siguientes parámetros:

- Punto de conexión de inquilino de Azure AD. La información del inquilino se puede recuperar desde Azure Portal. Mantenga el puntero sobre el usuario que inició sesión en la esquina superior derecha.
- URI del recurso de Media Services.
- Id. de cliente de aplicación de Media Services (nativo).
- URI de redireccionamiento de aplicación de Media Services (nativo).

Los valores para estos parámetros se pueden encontrar en **AzureEnvironments.AzureCloudEnvironment**. La constante **AzureEnvironments.AzureCloudEnvironment** es un asistente del SDK para .NET para obtener la configuración adecuada de la variable de entorno para un centro de datos de Azure público.

Contiene la configuración predefinida del entorno para acceder a Media Services solo en los centros de datos públicos. Para regiones con entornos de nube de administración pública o soberana, puede usar **AzureChinaCloudEnvironment**, **AzureUsGovernmentEnvironment** o **AzureGermanCloudEnvironment**, respectivamente.

En el ejemplo de código siguiente se crea un token:

```csharp
var tokenCredentials = new AzureAdTokenCredentials("microsoft.onmicrosoft.com", AzureEnvironments.AzureCloudEnvironment);
var tokenProvider = new AzureAdTokenProvider(tokenCredentials);
```

Para empezar a programar con Media Services, tiene que crear una instancia de **CloudMediaContext** que representa el contexto del servidor. **CloudMediaContext** incluye referencias a colecciones importantes, como trabajos, recursos, archivos, directivas de acceso y localizadores.

También necesita pasar el **URI de recurso de Media REST Services** al constructor **CloudMediaContext**. Para obtener el URI de recurso de Media REST Services, inicie sesión en Azure Portal, seleccione la cuenta de Azure Media Services, seleccione **Acceso de API** y después seleccione **Connect to Azure Media Services with user authentication** (Conexión a Azure Media Services con la autenticación de usuario).

En el ejemplo de código siguiente se crea una instancia de **CloudMediaContext**:

```csharp
CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);
```

En el ejemplo siguiente se muestra cómo crear el token de Azure AD y el contexto:

```csharp
namespace AzureADAuthSample
{
    class Program
    {
        static void Main(string[] args)
        {
            // Specify your Azure AD tenant domain, for example "microsoft.onmicrosoft.com".
            var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}", AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            // Specify your REST API endpoint, for example "https://accountname.restv2.westcentralus.media.azure.net/API".
            CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);

            var assets = context.Assets;
            foreach (var a in assets)
            {
                Console.WriteLine(a.Name);
            }
        }
    }
}
```

>[!NOTE]
>Si se produce una excepción que indica "El servidor remoto devolvió un error: (401)No autorizado", revise la sección [Control de acceso](media-services-use-aad-auth-to-access-ams-api.md#access-control) de Acceso a Azure Media Services API con la autenticación de Azure AD.

## <a name="use-service-principal-authentication"></a>Uso de la autenticación de entidad de servicio

Para conectarse a la API de Media Services mediante la opción de la entidad de servicio, la aplicación de nivel intermedio (Web API o aplicación web) necesita solicitar un token de Azure AD que tenga los parámetros siguientes:

- Punto de conexión de inquilino de Azure AD. La información del inquilino se puede recuperar desde Azure Portal. Mantenga el puntero sobre el usuario que inició sesión en la esquina superior derecha.
- URI del recurso de Media Services.
- Valores de aplicación de Azure AD: el **Id. de cliente** y el **secreto de cliente**.

Los valores para los parámetros **Id. de cliente** y **secreto de cliente** pueden encontrarse en Azure Portal. Para más información, vea [Introducción a la autenticación de Azure AD mediante Azure Portal](media-services-portal-get-started-with-aad.md).

El ejemplo de código siguiente crea un token con el uso del constructor **AzureAdTokenCredentials** que adopta **AzureAdClientSymmetricKey** como parámetro:

```csharp
var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}",
                            new AzureAdClientSymmetricKey("{YOUR CLIENT ID HERE}", "{YOUR CLIENT SECRET}"),
                            AzureEnvironments.AzureCloudEnvironment);

var tokenProvider = new AzureAdTokenProvider(tokenCredentials);
```

También puede especificar el constructor **AzureAdTokenCredentials** que adopta **AzureAdClientCertificate** como parámetro.

Para obtener instrucciones sobre cómo crear y configurar un certificado de una forma que Azure AD puede utilizar, vea [Authenticating to Azure AD in daemon apps with certificates - manual configuration steps](https://github.com/azure-samples/active-directory-dotnetcore-daemon-v2) (Autenticación en aplicaciones de demonio de Azure AD con certificados: pasos de configuración manual).

```csharp
var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}",
                            new AzureAdClientCertificate("{YOUR CLIENT ID HERE}", "{YOUR CLIENT CERTIFICATE THUMBPRINT}"),
                            AzureEnvironments.AzureCloudEnvironment);
```

Para empezar a programar con Media Services, tiene que crear una instancia de **CloudMediaContext** que representa el contexto del servidor. También necesita pasar el **URI de recurso de Media REST Services** al constructor **CloudMediaContext**. También puede obtener el valor del **URI de recurso para Media REST Services** en Azure Portal.

En el ejemplo de código siguiente se crea una instancia de **CloudMediaContext**:

```csharp
CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);
```

En el ejemplo siguiente se muestra cómo crear el token de Azure AD y el contexto:

```csharp
namespace AzureADAuthSample
{
    class Program
    {
        static void Main(string[] args)
        {
            var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}",
                                        new AzureAdClientSymmetricKey("{YOUR CLIENT ID HERE}", "{YOUR CLIENT SECRET}"),
                                        AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            // Specify your REST API endpoint, for example "https://accountname.restv2.westcentralus.media.azure.net/API".
            CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);

            var assets = context.Assets;
            foreach (var a in assets)
            {
                Console.WriteLine(a.Name);
            }

            Console.ReadLine();
        }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

Empiece a trabajar en la [carga de archivos en la cuenta](media-services-dotnet-upload-files.md).
