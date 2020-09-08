---
title: 'Conexión a la API de Azure Media Services v3: .NET'
description: En este artículo se muestra cómo conectarse a la API de Media Services v3 con .NET.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: has-adal-ref, devx-track-csharp
ms.openlocfilehash: b9b72e8340722f8ddc97b3769ce22e607974a508
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89297406"
---
# <a name="connect-to-media-services-v3-api---net"></a>Conexión a la API de Media Services v3: .NET

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En este artículo se muestra cómo conectar con el SDK de .NET de Azure Media Services v3 con el método de inicio de sesión de la entidad de servicio.

## <a name="prerequisites"></a>Prerrequisitos

- [Cree una cuenta de Media Services](./create-account-howto.md). Asegúrese de recordar el nombre del grupo de recursos y el nombre de la cuenta de Media Services.
- Instale una herramienta que quiera usar para el desarrollo con .NET. Los pasos descritos en este artículo muestran cómo usar [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). Puede usar Visual Studio Code, consulte [Working with C# ](https://code.visualstudio.com/docs/languages/csharp) (Trabajar con C#). O bien, puede usar otro editor de código.

> [!IMPORTANT]
> Revise las [convenciones de nomenclatura](media-services-apis-overview.md#naming-conventions).

## <a name="create-a-console-application"></a>Creación de una aplicación de consola

1. Inicie Visual Studio. 
1. En el menú **Archivo**, haga clic en **Nuevo** > **Proyecto**. 
1. Cree una aplicación de consola de **.NET Core**.

La aplicación de ejemplo de este tema tiene `netcoreapp2.0` como destino. El código usa "async main", disponible a partir de C# 7.1. Consulte este [blog](/archive/blogs/benwilli/async-main-is-available-but-hidden) para más información.

## <a name="add-required-nuget-packages"></a>Incorporación de los paquetes NuGet requeridos

1. En Visual Studio, seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes**.
2. En la ventana **Consola del Administrador de paquetes**, use el comando `Install-Package` para agregar los siguientes paquetes NuGet. Por ejemplo, `Install-Package Microsoft.Azure.Management.Media`.

|Paquete|Descripción|
|---|---|
|`Microsoft.Azure.Management.Media`|SDK de Azure Media Services. <br/>Para asegurarse de que usa el paquete más reciente de Azure Media Services, consulte [Microsoft.Azure.Management.Media](https://www.nuget.org/packages/Microsoft.Azure.Management.Media).|
|`Microsoft.Rest.ClientRuntime.Azure.Authentication`|Biblioteca con autenticación ADAL para el SDK de Azure SDK para .NET|
|`Microsoft.Extensions.Configuration.EnvironmentVariables`|Lectura de los valores de configuración de las variables de entorno y los archivos JSON locales|
|`Microsoft.Extensions.Configuration.Json`|Lectura de los valores de configuración de las variables de entorno y los archivos JSON locales
|`WindowsAzure.Storage`|SDK de Storage|

## <a name="create-and-configure-the-app-settings-file"></a>Creación y configuración del archivo de configuración de la aplicación

### <a name="create-appsettingsjson"></a>Creación de appsettings.json

1. Vaya a **General** > **Archivo de texto**.
1. Dele el nombre "appsettings.json".
1. Establezca la propiedad "Copy to Output Directory" del archivo .json en "Copy if newer" (para que la aplicación pueda acceder a ella al publicar).

### <a name="set-values-in-appsettingsjson"></a>Establecimiento de los valores en appsettings.json

Ejecute el comando `az ams account sp create` como se describe en el artículo de [acceso a las API](./access-api-howto.md). El comando devuelve el json que se debe copiar en "appsettings.json".
 
## <a name="add-configuration-file"></a>Adición de archivo de configuración

Para mayor comodidad, agregue un archivo de configuración que se encargue de leer los valores de "appsettings.json".

1. Agregue una nueva clase .cs al proyecto. Asígnele el nombre `ConfigWrapper`. 
1. Pegue el siguiente código en este archivo (en este ejemplo se supone que tiene el espacio de nombres `ConsoleApp1`).

```csharp
using System;

using Microsoft.Extensions.Configuration;

namespace ConsoleApp1
{
    public class ConfigWrapper
    {
        private readonly IConfiguration _config;

        public ConfigWrapper(IConfiguration config)
        {
            _config = config;
        }

        public string SubscriptionId
        {
            get { return _config["SubscriptionId"]; }
        }

        public string ResourceGroup
        {
            get { return _config["ResourceGroup"]; }
        }

        public string AccountName
        {
            get { return _config["AccountName"]; }
        }

        public string AadTenantId
        {
            get { return _config["AadTenantId"]; }
        }

        public string AadClientId
        {
            get { return _config["AadClientId"]; }
        }

        public string AadSecret
        {
            get { return _config["AadSecret"]; }
        }

        public Uri ArmAadAudience
        {
            get { return new Uri(_config["ArmAadAudience"]); }
        }

        public Uri AadEndpoint
        {
            get { return new Uri(_config["AadEndpoint"]); }
        }

        public Uri ArmEndpoint
        {
            get { return new Uri(_config["ArmEndpoint"]); }
        }

        public string Region
        {
            get { return _config["Region"]; }
        }
    }
}
```

## <a name="connect-to-the-net-client"></a>Conexión con el cliente de .NET

Para empezar a usar las API de Media Services con. NET, debe crear un objeto **AzureMediaServicesClient**. Para crear el objeto, debe proporcionar las credenciales necesarias para que el cliente se conecte a Azure mediante Azure AD. En el código siguiente, la función GetCredentialsAsync crea el objeto ServiceClientCredentials basándose en las credenciales proporcionadas en el archivo de configuración local.

1. Abra `Program.cs`.
1. Pegue el código siguiente:

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading.Tasks;

using Microsoft.Azure.Management.Media;
using Microsoft.Azure.Management.Media.Models;
using Microsoft.Extensions.Configuration;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.Rest;
using Microsoft.Rest.Azure.Authentication;

namespace ConsoleApp1
{
    class Program
    {
        public static async Task Main(string[] args)
        {
            
            ConfigWrapper config = new ConfigWrapper(new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddEnvironmentVariables()
                .Build());

            try
            {
                IAzureMediaServicesClient client = await CreateMediaServicesClientAsync(config);
                Console.WriteLine("connected");
            }
            catch (Exception exception)
            {
                if (exception.Source.Contains("ActiveDirectory"))
                {
                    Console.Error.WriteLine("TIP: Make sure that you have filled out the appsettings.json file before running this sample.");
                }

                Console.Error.WriteLine($"{exception.Message}");

                ApiErrorException apiException = exception.GetBaseException() as ApiErrorException;
                if (apiException != null)
                {
                    Console.Error.WriteLine(
                        $"ERROR: API call failed with error code '{apiException.Body.Error.Code}' and message '{apiException.Body.Error.Message}'.");
                }
            }

            Console.WriteLine("Press Enter to continue.");
            Console.ReadLine();
        }
 
        private static async Task<ServiceClientCredentials> GetCredentialsAsync(ConfigWrapper config)
        {
            // Use ApplicationTokenProvider.LoginSilentWithCertificateAsync or UserTokenProvider.LoginSilentAsync to get a token using service principal with certificate
            //// ClientAssertionCertificate
            //// ApplicationTokenProvider.LoginSilentWithCertificateAsync

            // Use ApplicationTokenProvider.LoginSilentAsync to get a token using a service principal with symmetric key
            ClientCredential clientCredential = new ClientCredential(config.AadClientId, config.AadSecret);
            return await ApplicationTokenProvider.LoginSilentAsync(config.AadTenantId, clientCredential, ActiveDirectoryServiceSettings.Azure);
        }

        private static async Task<IAzureMediaServicesClient> CreateMediaServicesClientAsync(ConfigWrapper config)
        {
            var credentials = await GetCredentialsAsync(config);

            return new AzureMediaServicesClient(config.ArmEndpoint, credentials)
            {
                SubscriptionId = config.SubscriptionId,
            };
        }

    }
}
```

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Carga, codificación y streaming de vídeos: .NET](stream-files-tutorial-with-api.md) 
- [Tutorial: Streaming en vivo con Media Services v3: .NET](stream-live-tutorial-with-api.md)
- [Tutorial: Análisis de vídeos con Media Services v3: .NET](analyze-videos-tutorial-with-api.md)
- [Creación de una entrada de trabajo a partir de un archivo local: .NET](job-input-from-local-file-how-to.md)
- [Creación de una entrada de trabajo a partir de una dirección URL de HTTPS: .NET](job-input-from-http-how-to.md)
- [Codificación con una transformación personalizada: .NET](customize-encoder-presets-how-to.md)
- [Uso del cifrado dinámico AES-128 y el servicio de entrega de claves: .NET](protect-with-aes128.md)
- [Uso del cifrado dinámico de DRM y el servicio de entrega de licencias: .NET](protect-with-drm.md)
- [Obtención de una clave de firma de la directiva existente: .NET](get-content-key-policy-dotnet-howto.md)
- [Creación de filtros con Media Services: .NET](filters-dynamic-manifest-dotnet-howto.md)
- [Ejemplos avanzados de vídeo bajo demanda de Azure Functions v2 con Media Services v3](https://aka.ms/ams3functions)

## <a name="see-also"></a>Consulte también

* [Referencia de .NET](/dotnet/api/overview/azure/mediaservices/management?view=azure-dotnet)
* Para obtener más ejemplos de código, consulte el repositorio de [ejemplos del SDK de .NET](https://github.com/Azure-Samples/media-services-v3-dotnet).
