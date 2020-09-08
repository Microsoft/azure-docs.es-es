---
title: 'Creación de revisiones de vídeo con .NET: Content Moderator'
titleSuffix: Azure Cognitive Services
description: En este artículo se proporciona información y ejemplos de código que le ayudarán a empezar a usar el SDK de Content Moderator con C# para crear revisiones de vídeos.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 10/24/2019
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: b2fb06c838de480bb73501307ab11cb3d6831921
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88919325"
---
# <a name="create-video-reviews-using-net"></a>Creación de revisiones de vídeo con .NET

En este artículo se proporciona información y ejemplos de código que le ayudarán a empezar a usar el SDK de [Content Moderator con C#](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) para:

- Crear una revisión de vídeo para moderadores humanos
- Adición de fotogramas a una revisión
- Obtención de fotogramas para la revisión
- Obtención del estado y los detalles de la revisión
- Publicar la revisión

## <a name="prerequisites"></a>Prerequisites

- Inicie sesión o cree una cuenta en el sitio de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/) de Content Moderator.
- En este artículo se da por supuesto que ha [moderado el vídeo (consulte la guía de inicio rápido)](video-moderation-api.md) y que tiene los datos de respuesta. Se necesita para crear revisiones basadas en fotogramas para moderadores humanos.

## <a name="ensure-your-api-key-can-call-the-review-api-for-review-creation"></a>Garantizar que la clave de API puede llamar a la API de revisión para la creación de revisiones

Tras completar los pasos anteriores, puede acabar con dos claves de Content Moderator si ha empezado desde Azure Portal.

Si tiene pensado usar la clave de API proporcionada por Azure en el ejemplo del SDK, siga los pasos mencionados en la sección [Uso de la clave de Azure con la API de revisión](review-tool-user-guide/configure.md#use-your-azure-account-with-the-review-apis) para permitir que la aplicación llame a la API de revisión y cree revisiones.

Si usa la clave de prueba gratuita generada por la herramienta de revisión, la cuenta de la herramienta de revisión ya conoce la clave y, por lo tanto, no se requiere ningún paso adicional.

### <a name="prepare-your-video-and-the-video-frames-for-review"></a>Preparación del vídeo y los fotogramas de vídeo para su revisión

El vídeo y los fotogramas de vídeo para revisar deben estar publicados en línea porque necesita sus direcciones URL.

> [!NOTE]
> El programa usa las capturas de pantalla del vídeo guardadas manualmente con puntuaciones aleatorias sobre el contenido para adultos o subido de tono para ilustrar el uso de la API de revisión. En una situación real, usaría la [salida de la moderación de vídeo](video-moderation-api.md#run-the-program-and-review-the-output) para crear imágenes y asignar las puntuaciones. 

Para ver el vídeo, necesita un punto de conexión de streaming para que la herramienta de revisión reproduzca el vídeo en la vista del reproductor.

![Miniatura de la demostración de vídeo](images/ams-video-demo-view.PNG)

- Copie la **Dirección URL** en la página de la [demostración de Azure Media Services](https://aka.ms/azuremediaplayer?url=https%3A%2F%2Famssamples.streaming.mediaservices.windows.net%2F91492735-c523-432b-ba01-faba6c2206a2%2FAzureMediaServicesPromo.ism%2Fmanifest) para la dirección URL del manifiesto.

Para los fotogramas de vídeo (imágenes), utilice las siguientes imágenes:

![Miniatura de fotograma de vídeo 1](images/ams-video-frame-thumbnails-1.PNG) | ![Miniatura de fotograma de vídeo 2](images/ams-video-frame-thumbnails-2.PNG) | ![Miniatura de fotograma de vídeo 3](images/ams-video-frame-thumbnails-3.PNG) |
| :---: | :---: | :---: |
[Fotograma 1](https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame1-00-17.PNG) | [Fotograma 2](https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-2-01-04.PNG) | [Fotograma 3](https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-3-02-24.PNG) |

## <a name="create-your-visual-studio-project"></a>Crear un proyecto de Visual Studio

1. Agregue un nuevo proyecto de **Aplicación de consola (.NET Framework)** a la solución.

1. Asigne el nombre **VideoReviews** al proyecto.

1. Seleccione este proyecto como proyecto de inicio único para la solución.

### <a name="install-required-packages"></a>Instalación de los paquetes requeridos

Instale los siguientes paquetes NuGet para el proyecto TermLists.

- Microsoft.Azure.CognitiveServices.ContentModerator
- Microsoft.Rest.ClientRuntime
- Microsoft.Rest.ClientRuntime.Azure
- Newtonsoft.Json

### <a name="update-the-programs-using-statements"></a>Actualizar las instrucciones using del programa

Modifique las instrucciones using de la siguiente manera.

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Threading;
using Microsoft.Azure.CognitiveServices.ContentModerator;
using Microsoft.Azure.CognitiveServices.ContentModerator.Models;
using Newtonsoft.Json;
```

### <a name="add-private-properties"></a>Adición de propiedades privadas

Agregue las siguientes propiedades privadas al espacio de nombres **VideoReviews**, clase **Program**. Actualice los campos `AzureEndpoint` y `CMSubscriptionKey` con los valores de la dirección URL del punto de conexión y la clave de suscripción. Puede encontrarlos en la pestaña **Inicio rápido** del recurso en Azure Portal.


```csharp
namespace VideoReviews
{
    class Program
    {
        // NOTE: Enter a valid endpoint URL
        /// <summary>
        /// The endpoint URL of your subscription
        /// </summary>
        private static readonly string AzureEndpoint = "YOUR ENDPOINT URL";

        // NOTE: Enter a valid subscription key.
        /// <summary>
        /// Your Content Moderator subscription key.
        /// </summary>
        private static readonly string CMSubscriptionKey = "YOUR CONTENT MODERATOR KEY";

        // NOTE: Replace this example team name with your Content Moderator team name.
        /// <summary>
        /// The name of the team to assign the job to.
        /// </summary>
        /// <remarks>This must be the team name you used to create your 
        /// Content Moderator account. You can retrieve your team name from
        /// the Content Moderator web site. Your team name is the Id associated 
        /// with your subscription.</remarks>
        private const string TeamName = "YOUR CONTENT MODERATOR TEAM ID";

        /// <summary>
        /// The minimum amount of time, in milliseconds, to wait between calls
        /// to the Content Moderator APIs.
        /// </summary>
        private const int throttleRate = 2000;
```

### <a name="create-content-moderator-client-object"></a>Creación de objeto cliente de Content Moderator

Agregue la siguiente definición de método al espacio de nombres **VideoReviews**, clase **Program**.

```csharp
/// <summary>
/// Returns a new Content Moderator client for your subscription.
/// </summary>
/// <returns>The new client.</returns>
/// <remarks>The <see cref="ContentModeratorClient"/> is disposable.
/// When you have finished using the client,
/// you should dispose of it either directly or indirectly. </remarks>
public static ContentModeratorClient NewClient()
{
    return new ContentModeratorClient(new ApiKeyServiceClientCredentials(CMSubscriptionKey))
    {
        Endpoint = AzureEndpoint
    };
}
```

## <a name="create-a-video-review"></a>Creación de una revisión de vídeo

Cree una revisión del vídeo con **ContentModeratorClient.Reviews.CreateVideoReviews**. Para más información, consulte la [referencia de API](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c4).

**CreateVideoReviews** tiene los siguientes parámetros requeridos:
1. Una cadena que contiene un tipo MIME, que debe ser "application/json". 
1. El nombre del equipo de Content Moderator.
1. Un objeto **IList\<CreateVideoReviewsBodyItem>**. Cada objeto **CreateVideoReviewsBodyItem** representa una revisión del vídeo. Esta guía de inicio rápido crea una revisión a la vez.

**CreateVideoReviewsBodyItem** tiene varias propiedades. Como mínimo, establezca las propiedades siguientes:
- **Content**. La dirección URL del vídeo para revisar.
- **ContentId**. Un identificador que se asigna a la revisión del vídeo.
- **Status**. Establezca el valor en "Sin publicar" Si no se establece, el valor predeterminado es "Pendiente", lo que significa que se publica la revisión del vídeo y queda pendiente de revisión humana. Una vez que se publique una revisión del vídeo, ya no se le podrá agregar fotogramas de vídeo, una transcripción o un resultado de moderación de la transcripción.

> [!NOTE]
> **CreateVideoReviews** devuelve un valor IList\<string>. Cada una de estas cadenas contiene un identificador para una revisión de vídeo. Estos identificadores son GUID y no son los mismos que el valor de la propiedad **ContentId**. 

Agregue la siguiente definición de método al espacio de nombres VideoReviews, clase Program.

```csharp
/// <summary>
/// Create a video review. For more information, see the API reference:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c4 
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="id">The ID to assign to the video review.</param>
/// <param name="content">The URL of the video to review.</param>
/// <returns>The ID of the video review.</returns>
private static string CreateReview(ContentModeratorClient client, string id, string content)
{
    Console.WriteLine("Creating a video review.");

    List<CreateVideoReviewsBodyItem> body = new List<CreateVideoReviewsBodyItem>() {
        new CreateVideoReviewsBodyItem
        {
            Content = content,
            ContentId = id,
            /* Note: to create a published review, set the Status to "Pending".
            However, you cannot add video frames or a transcript to a published review. */
            Status = "Unpublished",
        }
    };

    var result = client.Reviews.CreateVideoReviews("application/json", TeamName, body);

    Thread.Sleep(throttleRate);

    // We created only one review.
    return result[0];
}
```

> [!NOTE]
> La clave de servicio de Content Moderator tiene un límite de solicitudes por segundo (RPS) y, si se supera el límite, el SDK emite una excepción con un código de error 429.
>
> Una clave de un plan gratuito tiene un límite de una solicitud por segundo.

## <a name="add-video-frames-to-the-video-review"></a>Adición de fotogramas de vídeo a la revisión de vídeo

Agregue fotogramas de vídeo a una revisión de vídeo con **ContentModeratorClient.Reviews.AddVideoFrameUrl** (si los fotogramas de vídeo se hospedan en línea) o **ContentModeratorClient.Reviews.AddVideoFrameStream** (si los fotogramas de vídeo se hospedan localmente). En esta guía de inicio rápido se da por supuesto que los fotogramas de vídeo se hospedan en línea y, por lo tanto, se usa **AddVideoFrameUrl**. Para más información, consulte la [referencia de API](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7b76ae7151f0b10d451fd).

**AddVideoFrameUrl** tiene los siguientes parámetros requeridos:
1. Una cadena que contiene un tipo MIME, que debe ser "application/json".
1. El nombre del equipo de Content Moderator.
1. Identificador de revisión de vídeo que **CreateVideoReviews** devuelve.
1. Un objeto **IList\<VideoFrameBodyItem>**. Cada objeto **VideoFrameBodyItem** representa un fotograma de vídeo.

**VideoFrameBodyItem** tiene las siguientes propiedades:
- **Timestamp**. Cadena que contiene, en segundos, el tiempo en el vídeo a partir del cuál se tomó el fotograma de vídeo.
- **FrameImage**. Dirección URL del fotograma de vídeo.
- **Metadata**. Valor de IList\<VideoFrameBodyItemMetadataItem>. **VideoFrameBodyItemMetadataItem** es simplemente un par de clave-valor. Las claves válidas son:
- **reviewRecommended**. Es True si se recomienda la revisión humana del fotograma de vídeo.
- **adultScore**. Valor de 0 a 1 que clasifica la gravedad de contenido para adultos en el fotograma de vídeo.
- **a**. Es True si el vídeo incluye contenido para adultos.
- **racyScore**. Valor de 0 a 1 que clasifica la gravedad de contenido subido de tono en el fotograma de vídeo.
- **r**. Es True si el fotograma de vídeo incluye contenido subido de tono.
- **ReviewerResultTags**. Valor de IList\<VideoFrameBodyItemReviewerResultTagsItem>. **VideoFrameBodyItemReviewerResultTagsItem** es simplemente un par de clave-valor. Una aplicación puede utilizar estas etiquetas para organizar los fotogramas de vídeo.

> [!NOTE]
> Esta guía de inicio rápido genera valores aleatorios para las propiedades **adultScore** y **racyScore**. En una aplicación de producción, estos valores se obtienen en el [servicio de moderación de vídeo](video-moderation-api.md), implementado como un servicio de Azure Media Services.

Agregue las siguientes definiciones de método al espacio de nombres VideoReviews, clase Program.

```csharp
<summary>
/// Create a video frame to add to a video review after the video review is created.
/// </summary>
/// <param name="url">The URL of the video frame image.</param>
/// <returns>The video frame.</returns>
private static VideoFrameBodyItem CreateFrameToAddToReview(string url, string timestamp_seconds)
{
    // We generate random "adult" and "racy" scores for the video frame.
    Random rand = new Random();

    var frame = new VideoFrameBodyItem
    {
        // The timestamp is measured in milliseconds. Convert from seconds.
        Timestamp = (int.Parse(timestamp_seconds) * 1000).ToString(),
        FrameImage = url,

        Metadata = new List<VideoFrameBodyItemMetadataItem>
        {
            new VideoFrameBodyItemMetadataItem("reviewRecommended", "true"),
            new VideoFrameBodyItemMetadataItem("adultScore", rand.NextDouble().ToString()),
            new VideoFrameBodyItemMetadataItem("a", "false"),
            new VideoFrameBodyItemMetadataItem("racyScore", rand.NextDouble().ToString()),
            new VideoFrameBodyItemMetadataItem("r", "false")
        },

        ReviewerResultTags = new List<VideoFrameBodyItemReviewerResultTagsItem>()
        {
            new VideoFrameBodyItemReviewerResultTagsItem("tag1", "value1")
        }
    };

    return frame;
}
```

```csharp
/// <summary>
/// Add a video frame to the indicated video review. For more information, see the API reference:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7b76ae7151f0b10d451fd
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="review_id">The video review ID.</param>
/// <param name="url">The URL of the video frame image.</param>
static void AddFrame(ContentModeratorClient client, string review_id, string url, string timestamp_seconds)
{
    Console.WriteLine("Adding a frame to the review with ID {0}.", review_id);

    var frames = new List<VideoFrameBodyItem>()
    {
        CreateFrameToAddToReview(url, timestamp_seconds)
    };
        
    client.Reviews.AddVideoFrameUrl("application/json", TeamName, review_id, frames);

    Thread.Sleep(throttleRate);
```

## <a name="get-video-frames-for-video-review"></a>Obtención de los fotogramas de vídeo para la revisión de vídeo

Puede obtener los fotogramas de vídeo para una revisión de vídeo con **ContentModeratorClient.Reviews.GetVideoFrames**. **GetVideoFrames** tiene los siguientes parámetros requeridos:
1. El nombre del equipo de Content Moderator.
1. Identificador de revisión de vídeo que **CreateVideoReviews** devuelve.
1. Índice en base cero del primer fotograma de vídeo que se va a obtener.
1. Número de fotogramas que se va a obtener.

Agregue la siguiente definición de método al espacio de nombres VideoReviews, clase Program.

```csharp
/// <summary>
/// Get the video frames assigned to the indicated video review.  For more information, see the API reference:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7ba43e7151f0b10d45200
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="review_id">The video review ID.</param>
static void GetFrames(ContentModeratorClient client, string review_id)
{
    Console.WriteLine("Getting frames for the review with ID {0}.", review_id);

    Frames result = client.Reviews.GetVideoFrames(TeamName, review_id, 0);
    Console.WriteLine(JsonConvert.SerializeObject(result, Formatting.Indented));

    Thread.Sleep(throttleRate);
}
```

## <a name="get-video-review-information"></a>Obtención de la información de revisión de vídeo

La información de una revisión de vídeo se obtiene con **ContentModeratorClient.Reviews.GetReview**. **GetReview** tiene los siguientes parámetros requeridos:
1. El nombre del equipo de Content Moderator.
1. Identificador de revisión de vídeo que **CreateVideoReviews** devuelve.

Agregue la siguiente definición de método al espacio de nombres VideoReviews, clase Program.

```csharp
/// <summary>
/// Get the information for the indicated video review. For more information, see the reference API:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c2
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="review_id">The video review ID.</param>
private static void GetReview(ContentModeratorClient client, string review_id)
{
    Console.WriteLine("Getting the status for the review with ID {0}.", review_id);

    var result = client.Reviews.GetReview(ModeratorHelper.Clients.TeamName, review_id);
    Console.WriteLine(JsonConvert.SerializeObject(result, Formatting.Indented));

    Thread.Sleep(throttleRate);
}
```

## <a name="publish-video-review"></a>Publicación de revisión de vídeo

Una revisión de vídeo se publica con **ContentModeratorClient.Reviews.PublishVideoReview**. **PublishVideoReview** tiene los siguientes parámetros requeridos:
1. El nombre del equipo de Content Moderator.
1. Identificador de revisión de vídeo que **CreateVideoReviews** devuelve.

Agregue la siguiente definición de método al espacio de nombres VideoReviews, clase Program.

```csharp
/// <summary>
/// Publish the indicated video review. For more information, see the reference API:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7bb29e7151f0b10d45201
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="review_id">The video review ID.</param>
private static void PublishReview(ContentModeratorClient client, string review_id)
{
    Console.WriteLine("Publishing the review with ID {0}.", review_id);
    client.Reviews.PublishVideoReview(TeamName, review_id);
    Thread.Sleep(throttleRate);
}
```

## <a name="putting-it-all-together"></a>Resumen

Agregue la definición de método **Main** al espacio de nombres VideoReviews, clase Program. Por último, cierre la clase Program y el espacio de nombres VideoReviews.

```csharp
static void Main(string[] args)
{
    using (ContentModeratorClient client = NewClient())
    {
        // Create a review with the content pointing to a streaming endpoint (manifest)
        var streamingcontent = "https://amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest";
        string review_id = CreateReview(client, "review1", streamingcontent);

        var frame1_url = "https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame1-00-17.PNG";
        var frame2_url = "https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-2-01-04.PNG";
        var frame3_url = "https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-3-02-24.PNG";

        // Add the frames from 17, 64, and 144 seconds.
        AddFrame(client, review_id, frame1_url, "17");
        AddFrame(client, review_id, frame2_url, "64");
        AddFrame(client, review_id, frame3_url, "144");

        // Get frames information and show
        GetFrames(client, review_id);
        GetReview(client, review_id);

        // Publish the review
        PublishReview(client, review_id);

        Console.WriteLine("Open your Content Moderator Dashboard and select Review > Video to see the review.");
        Console.WriteLine("Press any key to close the application.");
        Console.ReadKey();
    }
}
```

## <a name="run-the-program-and-review-the-output"></a>Ejecutar el programa y revisar la salida
Al ejecutar la aplicación, verá una salida en las siguientes líneas:

```json
Creating a video review.
Adding a frame to the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
Adding a frame to the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
Adding a frame to the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
Getting frames for the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
{
    "ReviewId": "201801v3212bda70ced4928b2cd7459c290c7dc",
    "VideoFrames": [
    {
        "Timestamp": "17000",
        "FrameImage": "https://reviewcontentprod.blob.core.windows.net/testreview6/FRM_201801v3212bda70ced4928b2cd7459c290c7dc_17000.PNG",
        "Metadata": [
        {
            "Key": "reviewRecommended",
            "Value": "true"
        },
        {
            "Key": "adultScore",
            "Value": "0.808312381528463"
        },
        {
            "Key": "a",
            "Value": "false"
        },
        {
            "Key": "racyScore",
            "Value": "0.846378884206702"
        },
        {
            "Key": "r",
            "Value": "false"
        }
        ],
        "ReviewerResultTags": [
        {
            "Key": "tag1",
            "Value": "value1"
        }
    ]
    },
    {
        "Timestamp": "64000",
        "FrameImage": "https://reviewcontentprod.blob.core.windows.net/testreview6/FRM_201801v3212bda70ced4928b2cd7459c290c7dc_64000.PNG",
        "Metadata": [
        {
            "Key": "reviewRecommended",
            "Value": "true"
        },
        {
            "Key": "adultScore",
            "Value": "0.576078300166912"
        },
        {
            "Key": "a",
            "Value": "false"
        },
        {
            "Key": "racyScore",
            "Value": "0.244768953064815"
        },
        {
            "Key": "r",
            "Value": "false"
        }
        ],
        "ReviewerResultTags": [
        {
            "Key": "tag1",
            "Value": "value1"
        }
    ]
    },
    {
        "Timestamp": "144000",
        "FrameImage": "https://reviewcontentprod.blob.core.windows.net/testreview6/FRM_201801v3212bda70ced4928b2cd7459c290c7dc_144000.PNG",
        "Metadata": [
        {
            "Key": "reviewRecommended",
            "Value": "true"
        },
        {
            "Key": "adultScore",
            "Value": "0.664480847150311"
        },
        {
            "Key": "a",
            "Value": "false"
        },
        {
            "Key": "racyScore",
            "Value": "0.933817870418456"
        },
        {
            "Key": "r",
            "Value": "false"
        }
        ],
        "ReviewerResultTags": [
        {
            "Key": "tag1",
            "Value": "value1"
        }
        ]
    }
    ]
}

Getting the status for the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
{
    "ReviewId": "201801v3212bda70ced4928b2cd7459c290c7dc",
    "SubTeam": "public",
    "Status": "UnPublished",
    "ReviewerResultTags": [],
    "CreatedBy": "testreview6",
    "Metadata": [
    {
        "Key": "FrameCount",
        "Value": "3"
    }
    ],
    "Type": "Video",
    "Content": "https://amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest",
    "ContentId": "review1",
    "CallbackEndpoint": null
}

Publishing the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
Open your Content Moderator Dashboard and select Review > Video to see the review.
Press any key to close the application.
```

## <a name="check-out-your-video-review"></a>Comprobación de la revisión de vídeo

Por último, consulte la revisión de vídeo en su herramienta de revisión Content Moderator, en la pantalla **Review**>**Video** (Revisar > Vídeo).

![Revisión de vídeo para moderadores humanos](images/ams-video-review.PNG)

## <a name="next-steps"></a>Pasos siguientes

Obtenga el [SDK de Content Moderator para .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) y la [solución de Visual Studio](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator) para esta y otras guías de inicio rápido de Content Moderator para .NET.

Vea cómo agregar [moderación de transcripciones](video-transcript-moderation-review-tutorial-dotnet.md) a la revisión de vídeo. 

Consulte el tutorial detallado sobre cómo desarrollar una [solución completa de moderación de vídeo](video-transcript-moderation-review-tutorial-dotnet.md).
