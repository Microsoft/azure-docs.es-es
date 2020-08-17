---
title: Configuración del equipo para el desarrollo de Media Services con .NET
description: Conozca los requisitos previos de Media Services usando el SDK de Media Services para .NET. Aprenda también a crear una aplicación de Visual Studio.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: ec2804c7-c656-4fbf-b3e4-3f0f78599a7f
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 3e4d56698ed14ec260838f3ecbec91d58945e4ca
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87042942"
---
# <a name="media-services-development-with-net"></a>Desarrollo de Media Services con .NET 

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

En este artículo se explica cómo empezar a desarrollar aplicaciones de Media Services con .NET.

La **biblioteca del SDK de Azure Media Services para .NET** le permite programar en los Media Services mediante. NET. Para que el desarrollo con .NET sea aún más fácil, se proporciona la biblioteca **Extensiones del SDK de Azure Media Services para .NET**. Esta biblioteca contiene un conjunto de métodos de extensión y funciones auxiliares que simplifican el código de .NET. Las dos bibliotecas están disponibles a través de **NuGet** y **GitHub**.

## <a name="prerequisites"></a>Prerrequisitos
* Una cuenta de Media Services en una suscripción de Azure nueva o existente. Vea el artículo [Cómo crear una cuenta de Media Services](media-services-portal-create-account.md).
* Sistemas operativos: Windows 10, Windows 7, Windows 2008 R2 o Windows 8.
* .NET Framework 4.5 o superior.
* Visual Studio.

## <a name="create-and-configure-a-visual-studio-project"></a>Creación y configuración de un proyecto de Visual Studio
En esta sección se muestra cómo crear un proyecto en Visual Studio y configurarlo para el desarrollo de Media Services.  En este caso, el proyecto es una aplicación de consola Windows de C#, pero los pasos de configuración que se muestran aquí se aplican a otros tipos de proyectos que puede crear para las aplicaciones de Media Services (por ejemplo, una aplicación de Windows Forms o una aplicación web ASP.NET).

En esta sección se muestra cómo usar **NuGet** para agregar las extensiones del SDK de .NET para Media Services y otras bibliotecas dependientes.

También puede obtener los bits más recientes del SDK de Media Services para .NET en GitHub ([github.com/Azure/azure-sdk-for-media-services](https://github.com/Azure/azure-sdk-for-media-services) o [github.com/Azure/azure-sdk-for-media-services-extensions](https://github.com/Azure/azure-sdk-for-media-services-extensions)), compilar la solución y agregar las referencias al proyecto de cliente. Todas las dependencias necesarias se descargan y extraen automáticamente.

1. Cree una aplicación de consola en C# mediante Visual Studio. Escriba el **Nombre**, la **Ubicación** y el **Nombre de la solución** y, a continuación, haga clic en Aceptar.
2. Compile la solución.
3. Use **NuGet** para instalar y agregar **Extensiones del SDK de .NET para Azure Media Services** (**windowsazure.mediaservices.extensions**). Al instalar este paquete, también se instala el **SDK de Media Services para .NET** y agrega todas las demás dependencias necesarias.
   
    Asegúrese de que tiene instalada la versión más reciente de NuGet. Para más información e instrucciones de instalación, consulte [NuGet](https://nuget.codeplex.com/).

    1. En el Explorador de soluciones, haga clic con el botón derecho en el nombre del proyecto y elija **Administrar paquetes NuGet**.

    2. Aparecerá el cuadro de diálogo Administrar paquetes de NuGet.

    3. En la galería en línea, busque Extensiones de Azure Media Services, elija **Extensiones del SDK de .NET de Azure Media Services** (**windowsazure.mediaservices.extensions**) y haga clic en el botón **Instalar**.
   
    4. El proyecto se modifica y se agregan referencias a Extensiones del SDK de Media Services para .NET, al SDK de .NET de Media Services y a otros ensamblados dependientes.
4. Para promover un entorno de desarrollo más limpio, considere la posibilidad de habilitar la restauración de paquetes de NuGet. Para obtener más información, consulte [Restauración de paquetes de NuGet](https://docs.nuget.org/consume/package-restore).
5. Agregue una referencia al ensamblado **System.Configuration** . Este ensamblado contiene la clase de Configuración del sistema.**Administrador de configuración** que se usa para tener acceso a los archivos de configuración (por ejemplo, App.config).
   
    1. Para agregar referencias usando el cuadro de diálogo de administración de referencias, haga clic con el botón derecho en el nombre del proyecto en el Explorador de soluciones. Después haga clic en **Agregar** y en **Referencia**.
   
    2. Aparecerá el cuadro de diálogo Administrar referencias.
    3. En los ensamblados de .NET Framework, busque, seleccione el ensamblado System.Configuration y haga clic en **Aceptar**.
6. Abra el archivo App.config y agregue una sección **appSettings** al archivo. Establezca los valores que se necesitan para conectarse a la API de Media Services. Para más información, consulte [Acceso a Azure Media Services API con la autenticación de Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

    Establezca los valores que se necesitan para conectarse mediante el método de autenticación de **entidad de servicio**.

    ```xml
    <configuration>
    ...
        <appSettings>
            <add key="AMSAADTenantDomain" value="tenant"/>
            <add key="AMSRESTAPIEndpoint" value="endpoint"/>
            <add key="AMSClientId" value="id"/>
            <add key="AMSClientSecret" value="secret"/>
        </appSettings>
    </configuration>
    ```

7. Agregue la referencia **System.Configuration** al proyecto.
8. Sobrescriba las instrucciones **using** existentes siguiendo las instrucciones al comienzo del archivo Program.cs con el siguiente código:

    ```csharp      
    using System;
    using System.Configuration;
    using System.IO;
    using Microsoft.WindowsAzure.MediaServices.Client;
    using System.Threading;
    using System.Collections.Generic;
    using System.Linq;
    ```

    En este punto, está listo para iniciar el desarrollo de una aplicación de Media Services.    

## <a name="example"></a>Ejemplo

Este es un pequeño ejemplo en que se establece una conexión a la API de AMS y muestra todos los procesadores de multimedia disponibles.

```csharp
class Program
{
    // Read values from the App.config file.

    private static readonly string _AADTenantDomain =
        ConfigurationManager.AppSettings["AMSAADTenantDomain"];
    private static readonly string _RESTAPIEndpoint =
        ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
    private static readonly string _AMSClientId =
        ConfigurationManager.AppSettings["AMSClientId"];
    private static readonly string _AMSClientSecret =
        ConfigurationManager.AppSettings["AMSClientSecret"];
        
    private static CloudMediaContext _context = null;
    static void Main(string[] args)
    {
        AzureAdTokenCredentials tokenCredentials = 
            new AzureAdTokenCredentials(_AADTenantDomain,
                new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                AzureEnvironments.AzureCloudEnvironment);

        var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

        _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);
        
        // List all available Media Processors
        foreach (var mp in _context.MediaProcessors)
        {
            Console.WriteLine(mp.Name);
        }
        
    }
 ```

## <a name="next-steps"></a>Pasos siguientes

Ahora [puede conectarse a la API de AMS](media-services-use-aad-auth-to-access-ams-api.md) y empezar a [desarrollar](media-services-dotnet-get-started.md).


## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
