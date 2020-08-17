---
title: Introducción a la entrega de contenido a petición mediante .NET | Microsoft Docs
description: Este tutorial le guiará por los pasos necesarios para implementar una aplicación de entrega de contenido a petición con Azure Media Services mediante la API de REST.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 388b8928-9aa9-46b1-b60a-a918da75bd7b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 2c79566f51647492118f307e1c9737b98b1be484
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543595"
---
# <a name="get-started-with-delivering-content-on-demand-using-net-sdk"></a>Introducción a la entrega de contenido a petición mediante .NET SDK  

[!INCLUDE [media-services-selector-get-started](../../../includes/media-services-selector-get-started.md)]

Este tutorial le guiará por los pasos necesarios para implementar un servicio básico de entrega de contenido de vídeo bajo demanda (VoD) con la aplicación de Azure Media Services (AMS) mediante el SDK de .NET para Azure Media Services.

## <a name="prerequisites"></a>Prerrequisitos

Estos son los requisitos previos para completar el tutorial.

* Una cuenta de Azure. Para obtener más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/).
* Una cuenta de Media Services. Para crear una cuenta de Media Services, consulte el tema [Creación de una cuenta de Media Services](media-services-portal-create-account.md).
* .NET Framework 4.0 o posterior.
* Visual Studio.

Este tutorial incluye las siguientes tareas:

1. Inicio de punto de conexión de streaming (mediante Azure Portal).
2. Creación y configuración de un proyecto de Visual Studio.
3. Conexión a la cuenta de Media Services
2. Carga de un archivo de vídeo.
3. Codificación del archivo de origen en un conjunto de archivos MP4 de velocidad de bits adaptativa
4. Publicación del recurso y obtención de direcciones URL de descarga progresiva y streaming.  
5. Reproduzca el contenido.

## <a name="overview"></a>Información general
Este tutorial le guiará por los pasos necesarios para implementar una aplicación de entrega de contenido de vídeo a petición (VoD) con el SDK de Azure Media Services (AMS) para .NET.

Presenta el flujo de trabajo básico de Media Services y la mayoría de los objetos y tareas de programación más comunes necesarios para el desarrollo de Media Services. Al término del tutorial, podrá transmitir o cargar progresivamente un archivo multimedia de ejemplo que cargó, codificó y descargó.

### <a name="ams-model"></a>Modelo AMS

En la ilustración siguiente se muestran algunos de los objetos que se utilizan con más frecuencia al desarrollar aplicaciones de VoD con el modelo OData de Media Services.

Haga clic en la imagen para verla a tamaño completo.  

[![Diagrama que muestra algunos de los objetos que se usan con más frecuencia en el modelo de datos de objetos de Azure Media Services para desarrollar aplicaciones de vídeo bajo demanda.](./media/media-services-dotnet-get-started/media-services-overview-object-model-small.png)](./media/media-services-dotnet-get-started/media-services-overview-object-model.png#lightbox)

Puede ver el modelo completo [aquí](https://media.windows.net/API/$metadata?api-version=2.15).  

## <a name="start-streaming-endpoints-using-the-azure-portal"></a>Inicio de los puntos de conexión de streaming con Azure Portal

Cuando se trabaja con Azure Media Services, uno de los escenarios más comunes es entregar contenido de vídeo a los clientes mediante streaming con velocidad de bits adaptable. Media Services proporciona empaquetado dinámico que permite entregar contenido codificado MP4 con velocidad de bits adaptable en formatos de streaming admitidos por Media Services (MPEG DASH, HLS, Smooth Streaming) justo a tiempo, sin tener que almacenar versiones previamente empaquetadas de cada uno de estos formatos.

>[!NOTE]
>Cuando se crea la cuenta de AMS, se agrega un punto de conexión de streaming **predeterminado** a la cuenta en estado **Stopped** (Detenido). Para iniciar la transmisión del contenido y aprovechar el empaquetado dinámico y el cifrado dinámico, el punto de conexión de streaming desde el que va a transmitir el contenido debe estar en estado **Running** (En ejecución).

Para iniciar el punto de conexión de streaming, haga lo siguiente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En la ventana Settings (Configuración), haga clic en Streaming endpoints (Puntos de conexión de streaming).
3. Haga clic en el punto de conexión de streaming predeterminado.

    Aparecerá la ventana de detalles del punto de conexión de streaming predeterminado.

4. Haga clic en el icono de inicio.
5. Haga clic en el botón Save (Guardar) para guardar los cambios.

## <a name="create-and-configure-a-visual-studio-project"></a>Creación y configuración de un proyecto de Visual Studio

1. Configure el entorno de desarrollo y rellene el archivo app.config con la información de la conexión, como se describe en [Desarrollo de Media Services con .NET](media-services-dotnet-how-to-use.md). 
2. Cree una carpeta nueva (la carpeta puede estar en cualquier lugar del disco duro) y copie el archivo .mp4 o .wmv que desea codificar para reproducirlo en streaming o descargarlo progresivamente. En este ejemplo, se usa la ruta de acceso "C:\VideoFiles".

## <a name="connect-to-the-media-services-account"></a>Conexión a la cuenta de Media Services

Cuando se usa Media Services con .NET, debe usar la clase **CloudMediaContext** para la mayoría de las tareas de programación de Media Services: conexión a la cuenta de Media Services; creación, actualización, acceso y eliminación de los siguientes objetos: activos, archivos de activos, trabajos, directivas de acceso, localizadores, etc.

Sobrescriba la clase de programa predeterminada con el código siguiente: El código muestra cómo leer los valores de conexión del archivo App.config y cómo crear el objeto **CloudMediaContext** para conectarse a Media Services. Para más información, consulte [Conexión a la API de Media Services](media-services-use-aad-auth-to-access-ams-api.md).

No olvide actualizar el nombre de archivo y la ruta de acceso con la ubicación del archivo multimedia.

La función **Main** llama a métodos que se definirán más adelante en esta sección.

> [!NOTE]
> Recibirá errores de compilación hasta que agregue las definiciones para todas las funciones que se definen más adelante en este artículo.

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
        try
        {
            AzureAdTokenCredentials tokenCredentials = 
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            // Add calls to methods defined in this section.
            // Make sure to update the file name and path to where you have your media file.
            IAsset inputAsset =
            UploadFile(@"C:\VideoFiles\BigBuckBunny.mp4", AssetCreationOptions.None);

            IAsset encodedAsset =
            EncodeToAdaptiveBitrateMP4s(inputAsset, AssetCreationOptions.None);

            PublishAssetGetURLs(encodedAsset);
        }
        catch (Exception exception)
        {
            // Parse the XML error message in the Media Services response and create a new
            // exception with its content.
            exception = MediaServicesExceptionParser.Parse(exception);

            Console.Error.WriteLine(exception.Message);
        }
        finally
        {
            Console.ReadLine();
        }
        }
```

## <a name="create-a-new-asset-and-upload-a-video-file"></a>Creación de un nuevo recurso y carga de un archivo de vídeo

En Media Services, cargará (o ingerirá) los archivos digitales en un recurso. La entidad **Recurso** puede contener archivos de vídeo, audio, imágenes, colecciones de miniaturas, pistas de texto y subtítulos (y los metadatos acerca de estos archivos).  Una vez cargados los archivos, el contenido se almacena de forma segura en la nube para un posterior procesamiento y streaming. Los archivos del recurso se denominan **archivos de recursos**.

El método **UploadFile** definido a continuación llama a **CreateFromFile** (definido en las extensiones del SDK para .NET). **CreateFromFile** crea un nuevo recurso en el que se carga el archivo de origen especificado.

El método **CreateFromFile** toma **AssetCreationOptions**, que permite especificar una de las siguientes opciones de creación de recursos:

* **Ninguno** : no se utiliza cifrado. Este es el valor predeterminado. Tenga en cuenta que al usar esta opción el contenido no está protegido en tránsito o en reposo en el almacenamiento.
  Si tiene previsto entregar un MP4 mediante una descarga progresiva, utilice esta opción.
* **StorageEncrypted** : use esta opción para cifrar el contenido no cifrado de manera local mediante el cifrado Estándar de cifrado avanzado (AES) de 256 bits y luego cargarlo en Azure Storage donde se almacena cifrado en reposo. Los recursos protegidos con el cifrado de almacenamiento se descifran automáticamente y se colocan en un sistema de archivos cifrados antes de la codificación y, opcionalmente, se vuelven a cifrar antes de volver a cargarlos como un nuevo recurso de salida. El caso de uso principal para el cifrado de almacenamiento es cuando desea proteger los archivos multimedia de entrada de alta calidad con un sólido cifrado en reposo en disco.
* **CommonEncryptionProtected** : use esta opción si va a cargar contenido que ya se cifró y protegió con cifrado común o DRM de PlayReady (por ejemplo, Smooth Streaming protegido con DRM de PlayReady).
* **EnvelopeEncryptionProtected** : use esta opción si va a cargar HLS cifrado con AES. Tenga en cuenta que los archivos deben haberse codificado y cifrado con Transform Manager.

El método **CreateFromFile** también permite especificar una devolución de llamada para notificar el progreso de carga del archivo.

En el siguiente ejemplo especificamos **Ninguno** para las opciones del activo.

Agregue el método siguiente a la clase Program

```csharp
    static public IAsset UploadFile(string fileName, AssetCreationOptions options)
    {
        IAsset inputAsset = _context.Assets.CreateFromFile(
            fileName,
            options,
            (af, p) =>
            {
                Console.WriteLine("Uploading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
            });

        Console.WriteLine("Asset {0} created.", inputAsset.Id);

        return inputAsset;
    }
```

## <a name="encode-the-source-file-into-a-set-of-adaptive-bitrate-mp4-files"></a>Codificación del archivo de origen en un conjunto de archivos MP4 de velocidad de bits adaptativa
Después de especificar los recursos en Media Services, los elementos multimedia se pueden codificar, transmultiplexar, agregar una marca de agua, entre otras opciones, antes de entregarse a los clientes. Estas actividades se programan y se ejecutan en varias instancias de rol en segundo plano para garantizar la disponibilidad y alto rendimiento. Estas actividades se denominan trabajos y cada trabajo está compuesto de tareas atómicas que realizan el trabajo real en el archivo del recurso.

Como se ha indicado antes, cuando se trabaja con Azure Media Services, uno de los escenarios más comunes es ofrecer streaming con velocidad de bits adaptable a los clientes. Media Services puede empaquetar de manera dinámica un conjunto de archivos MP4 de velocidad de bits adaptable en uno de los siguientes formatos: HTTP Live Streaming (HLS), Smooth Streaming y MPEG DASH.

Para aprovechar el empaquetado dinámico, tiene que modificar o transcodificar el archivo intermedio (origen) en un conjunto de archivos MP4 de velocidad de bits adaptable o de Smooth Streaming de velocidad de bits adaptable.  

El código siguiente muestra cómo enviar un trabajo de codificación. El trabajo contiene una tarea que especifica la transcodificación del archivo intermedio en un conjunto de MP4 de velocidades de bits adaptables con el **Estándar de Codificador multimedia**. El código envía el trabajo y espera hasta que se complete.

Una vez completado el trabajo, debería poder transmitir el recurso o descargar progresivamente archivos MP4 creados como resultado de la transcodificación.

Agregue el método siguiente a la clase Program

```csharp
    static public IAsset EncodeToAdaptiveBitrateMP4s(IAsset asset, AssetCreationOptions options)
    {

        // Prepare a job with a single task to transcode the specified asset
        // into a multi-bitrate asset.

        IJob job = _context.Jobs.CreateWithSingleTask(
            "Media Encoder Standard",
            "Adaptive Streaming",
            asset,
            "Adaptive Bitrate MP4",
            options);

        Console.WriteLine("Submitting transcoding job...");


        // Submit the job and wait until it is completed.
        job.Submit();

        job = job.StartExecutionProgressTask(
            j =>
            {
                Console.WriteLine("Job state: {0}", j.State);
                Console.WriteLine("Job progress: {0:0.##}%", j.GetOverallProgress());
            },
            CancellationToken.None).Result;

        Console.WriteLine("Transcoding job finished.");

        IAsset outputAsset = job.OutputMediaAssets[0];

        return outputAsset;
    }
```

## <a name="publish-the-asset-and-get-urls-for-streaming-and-progressive-download"></a>Publicación del recurso y obtención de direcciones URL para streaming y descarga progresiva

Para transmitir o descargar un recurso, necesita "publicarlo" mediante la creación de un localizador. Los localizadores proporcionan acceso a los archivos contenidos en el recurso. Media Services admite dos tipos de localizadores: OnDemandOrigin, usados para transmitir contenido (por ejemplo, MPEG DASH, HLS o Smooth Streaming) y localizadores de firma de acceso (SAS), que se usan para descargar archivos multimedia.

### <a name="some-details-about-url-formats"></a>Algunos detalles sobre los formatos de direcciones URL

Una vez creados los localizadores, puede generar las direcciones URL que se van a utilizar para reproducir los archivos en streaming o descargarlos. En el ejemplo de este tutorial, la salida muestra direcciones URL que puede pegar en los exploradores correspondientes. En esta sección se incluyen breves ejemplos del aspecto de los diferentes formatos.

#### <a name="a-streaming-url-for-mpeg-dash-has-the-following-format"></a>Una dirección URL de streaming de MPEG DASH tiene el formato siguiente:

{nombre del punto de conexión de streaming-nombre de la cuenta de media services}.streaming.mediaservices.windows.net/{id. de localizador}/{nombre del archivo}.ism/Manifest **(format=mpd-time-csf)**

#### <a name="a-streaming-url-for-hls-has-the-following-format"></a>Una dirección URL de streaming de HLS tiene el formato siguiente:

{nombre del punto de conexión de streaming-nombre de la cuenta de media services}.streaming.mediaservices.windows.net/{id. de localizador}/{nombre del archivo}.ism/Manifest **(format=m3u8-aapl)**

#### <a name="a-streaming-url-for-smooth-streaming-has-the-following-format"></a>Una dirección URL de streaming de Smooth Streaming tiene el formato siguiente:

{nombre de extremo de streaming-nombre de cuenta de servicios multimedia}.streaming.mediaservices.windows.net/{Id. de localizador}/{filename}.ism/Manifest


#### <a name="a-sas-url-used-to-download-files-has-the-following-format"></a>Una dirección URL de SAS utilizada para descargar archivos tiene el formato siguiente:

{nombre del contenedor de blobs}/{nombre del recurso}/{nombre del archivo}/{firma de SAS}

Las extensiones de SDK de Media Services para .NET ofrecen útiles métodos auxiliares que devuelven direcciones URL con formato para el recurso publicado.

El código siguiente usa extensiones del SDK de .NET para crear localizadores y obtener las direcciones URL de streaming y descarga progresiva. El código también muestra cómo descargar los archivos en una carpeta local.

Agregue el método siguiente a la clase Program

```csharp
    static public void PublishAssetGetURLs(IAsset asset)
    {
        // Publish the output asset by creating an Origin locator for adaptive streaming,
        // and a SAS locator for progressive download.

        _context.Locators.Create(
            LocatorType.OnDemandOrigin,
            asset,
            AccessPermissions.Read,
            TimeSpan.FromDays(30));

        _context.Locators.Create(
            LocatorType.Sas,
            asset,
            AccessPermissions.Read,
            TimeSpan.FromDays(30));


        IEnumerable<IAssetFile> mp4AssetFiles = asset
                .AssetFiles
                .ToList()
                .Where(af => af.Name.EndsWith(".mp4", StringComparison.OrdinalIgnoreCase));

        // Get the Smooth Streaming, HLS and MPEG-DASH URLs for adaptive streaming,
        // and the Progressive Download URL.
        Uri smoothStreamingUri = asset.GetSmoothStreamingUri();
        Uri hlsUri = asset.GetHlsUri();
        Uri mpegDashUri = asset.GetMpegDashUri();

        // Get the URls for progressive download for each MP4 file that was generated as a result
        // of encoding.
        List<Uri> mp4ProgressiveDownloadUris = mp4AssetFiles.Select(af => af.GetSasUri()).ToList();


        // Display  the streaming URLs.
        Console.WriteLine("Use the following URLs for adaptive streaming: ");
        Console.WriteLine(smoothStreamingUri);
        Console.WriteLine(hlsUri);
        Console.WriteLine(mpegDashUri);
        Console.WriteLine();

        // Display the URLs for progressive download.
        Console.WriteLine("Use the following URLs for progressive download.");
        mp4ProgressiveDownloadUris.ForEach(uri => Console.WriteLine(uri + "\n"));
        Console.WriteLine();

        // Download the output asset to a local folder.
        string outputFolder = "job-output";
        if (!Directory.Exists(outputFolder))
        {
            Directory.CreateDirectory(outputFolder);
        }

        Console.WriteLine();
        Console.WriteLine("Downloading output asset files to a local folder...");
        asset.DownloadToFolder(
            outputFolder,
            (af, p) =>
            {
                Console.WriteLine("Downloading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
            });

        Console.WriteLine("Output asset files available at '{0}'.", Path.GetFullPath(outputFolder));
    }
```

## <a name="test-by-playing-your-content"></a>Prueba mediante la reproducción de contenido

Una vez ejecutado el programa definido en la sección anterior, se mostrarán las direcciones URL similares a la siguiente en la ventana de consola.

Direcciones URL de streaming adaptable:

Smooth Streaming

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest`

HLS

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=m3u8-aapl)`

MPEG DASH

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=mpd-time-csf)`

Direcciones URL de descarga progresiva (audio y vídeo).

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1500kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1000kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_56kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`


Para transmitir el vídeo, copie la dirección URL en el cuadro de texto URL de [Azure Media Services Player](https://aka.ms/azuremediaplayer).

Para probar la descarga progresiva, pegue una dirección URL en un explorador (por ejemplo, Internet Explorer, Chrome o Safari).

Para obtener más información, vea los temas siguientes:

- [Reproducir contenido con reproductores existentes](media-services-playback-content-with-existing-players.md)
- [Incrustación de un vídeo de streaming adaptativo MPEG-DASH en una aplicación HTML5 con DASH.js](media-services-embed-mpeg-dash-in-html5.md)

## <a name="download-sample"></a>Descarga de un ejemplo
El ejemplo siguiente contiene el código que creó en este tutorial: [ejemplo](https://azure.microsoft.com/documentation/samples/media-services-dotnet-on-demand-encoding-with-media-encoder-standard/).

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]



<!-- Anchors. -->


<!-- URLs. -->
[Web Platform Installer]: https://go.microsoft.com/fwlink/?linkid=255386
[Portal]: https://portal.azure.com/
