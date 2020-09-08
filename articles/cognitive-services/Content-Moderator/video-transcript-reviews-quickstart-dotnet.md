---
title: 'Creación de revisiones de transcripciones de vídeo con .NET: Content Moderator'
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo crear revisiones de transcripciones de vídeo con el SDK de Content Moderator de Az Cognitive Services para .NET
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 10/24/2019
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: f2d0ddae8a9bd8054c740402b8beb3bb0bccfa9f
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88919223"
---
# <a name="create-video-transcript-reviews-using-net"></a>Creación de revisiones de transcripciones de vídeo con .NET

En este artículo se proporciona información y ejemplos de código que le ayudarán a empezar a usar el SDK de [Content Moderator con C#](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) para:

- Crear una revisión de vídeo para moderadores humanos
- Agregar una transcripción moderada a la revisión
- Publicar la revisión

## <a name="prerequisites"></a>Prerequisites

- Inicie sesión en el sitio de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/) de Content Moderator, o cree una cuenta si aún no tiene una.
- En este artículo se da por hecho que ha [moderado el vídeo](video-moderation-api.md) y ha [creado la revisión de vídeo](video-reviews-quickstart-dotnet.md) en la herramienta de revisión para la toma de decisiones humana. Ahora desea agregar transcripciones de vídeo moderadas en la herramienta de revisión.

## <a name="ensure-your-api-key-can-call-the-review-api-job-creation"></a>Garantía de que la clave de API puede llamar a la API de revisión (creación de trabajo)

Tras completar los pasos anteriores, puede acabar con dos claves de Content Moderator si ha empezado desde Azure Portal.

Si tiene pensado usar la clave de API proporcionada por Azure en el ejemplo del SDK, siga los pasos mencionados en la sección [Uso de la clave de Azure con la API de revisión](./review-tool-user-guide/configure.md#use-your-azure-account-with-the-review-apis) para permitir que la aplicación llame a la API de revisión y cree revisiones.

Si usa la clave de prueba gratuita generada por la herramienta de revisión, la cuenta de la herramienta de revisión ya conoce la clave y, por lo tanto, no se requiere ningún paso adicional.

## <a name="prepare-your-video-for-review"></a>Preparación del vídeo para su revisión

Agregue la transcripción a una revisión de vídeo. El vídeo debe estar publicado en línea. Necesita su punto de conexión de streaming. El punto de conexión de streaming permite que el reproductor de vídeo de la herramienta de revisión reproduzca el vídeo.

![Miniatura de la demostración de vídeo](images/ams-video-demo-view.PNG)

- Copie la **Dirección URL** en la página de la [demostración de Azure Media Services](https://aka.ms/azuremediaplayer?url=https%3A%2F%2Famssamples.streaming.mediaservices.windows.net%2F91492735-c523-432b-ba01-faba6c2206a2%2FAzureMediaServicesPromo.ism%2Fmanifest) para la dirección URL del manifiesto.

## <a name="create-your-visual-studio-project"></a>Crear un proyecto de Visual Studio

1. Agregue un nuevo proyecto de **Aplicación de consola (.NET Framework)** a la solución.

1. Ponga al proyecto el nombre **VideoTranscriptReviews**.

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

Agregue las siguientes propiedades privadas al espacio de nombres **VideoTranscriptReviews**, clase **Program**. Actualice los campos `AzureEndpoint` y `CMSubscriptionKey` con los valores de la dirección URL del punto de conexión y la clave de suscripción. Puede encontrarlos en la pestaña **Inicio rápido** del recurso en Azure Portal.

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

Agregue la siguiente definición de método al espacio de nombres VideoTranscriptReviews, clase Program.

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
> La clave de servicio de Content Moderator tiene un límite de frecuencia de solicitudes por segundo (RPS). Si se supera el límite, el SDK produce una excepción con un código de error 429.
>
> Una clave de un plan gratuito tiene un límite de una solicitud por segundo.

## <a name="add-transcript-to-video-review"></a>Adición de una transcripción a una revisión de vídeo

Para agregar una transcripción a una revisión de vídeo, se utiliza **ContentModeratorClient.Reviews.AddVideoTranscript**. **AddVideoTranscript** tiene los siguientes parámetros requeridos:
1. El identificador del equipo de Content Moderator.
1. Identificador de revisión de vídeo que **CreateVideoReviews** devuelve.
1. Un objeto **Stream** que contiene la transcripción.

La transcripción debe tener el formato WebVTT. Para obtener más información, vea [Web Video Text Tracks Format](https://www.w3.org/TR/webvtt1/) (Formato de pistas de texto de vídeo web).

> [!NOTE]
> El programa utiliza una transcripción de ejemplo en formato VTT. En una solución real, utilice el servicio Azure Media Indexer para [generar una transcripción](https://docs.microsoft.com/azure/media-services/media-services-index-content) a partir de un vídeo.

Agregue la siguiente definición de método al espacio de nombres VideotranscriptReviews, clase Program.

```csharp
/// <summary>
/// Add a transcript to the indicated video review.
/// The transcript must be in the WebVTT format.
/// For more information, see the API reference:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7b8b2e7151f0b10d451fe
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="review_id">The video review ID.</param>
/// <param name="transcript">The video transcript.</param>
static void AddTranscript(ContentModeratorClient client, string review_id, string transcript)
{
    Console.WriteLine("Adding a transcript to the review with ID {0}.", review_id);
    client.Reviews.AddVideoTranscript(TeamName, review_id, new MemoryStream(System.Text.Encoding.UTF8.GetBytes(transcript)));
    Thread.Sleep(throttleRate);
}
```

## <a name="add-a-transcript-moderation-result-to-video-review"></a>Adición de un resultado de moderación de transcripción a la revisión de vídeo

Además de agregar una transcripción a una revisión de vídeo, también agregará el resultado de la moderación de dicha transcripción. Para ello, utiliza **ContentModeratorClient.Reviews.AddVideoTranscriptModerationResult**. Para más información, consulte la [referencia de API](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7b93ce7151f0b10d451ff).

**AddVideoTranscriptModerationResult** tiene los siguientes parámetros requeridos:
1. Una cadena que contiene un tipo MIME, que debe ser "application/json". 
1. El nombre del equipo de Content Moderator.
1. Identificador de revisión de vídeo que **CreateVideoReviews** devuelve.
1. Valor de IList\<TranscriptModerationBodyItem>. El elemento **TranscriptModerationBodyItem** tiene las siguientes propiedades:
1. **Terms**. Valor de IList\<TranscriptModerationBodyItemTermsItem>. El elemento **TranscriptModerationBodyItemTermsItem** tiene las siguientes propiedades:
1. **Index**. Índice de base cero del término.
1. **Term**. Una cadena que contiene el término.
1. **Timestamp**. Una cadena que contiene, en segundos, el tiempo en la transcripción en el que se encuentran los términos.

La transcripción debe tener el formato WebVTT. Para obtener más información, vea [Web Video Text Tracks Format](https://www.w3.org/TR/webvtt1/) (Formato de pistas de texto de vídeo web).

Agregue la siguiente definición de método al espacio de nombres VideoTranscriptReviews, clase Program. Este método envía una transcripción al método **ContentModeratorClient.TextModeration.ScreenText**. También convierte el resultado en un valor IList\<TranscriptModerationBodyItem> y lo envía a **AddVideoTranscriptModerationResult**.

```csharp
/// <summary>
/// Add the results of moderating a video transcript to the indicated video review.
/// For more information, see the API reference:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7b93ce7151f0b10d451ff
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="review_id">The video review ID.</param>
/// <param name="transcript">The video transcript.</param>
static void AddTranscriptModerationResult(ContentModeratorClient client, string review_id, string transcript)
{
    Console.WriteLine("Adding a transcript moderation result to the review with ID {0}.", review_id);

    // Screen the transcript using the Text Moderation API. For more information, see:
    // https://westus2.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66f
    Screen screen = client.TextModeration.ScreenText("eng", "text/plain", transcript);

    // Map the term list returned by ScreenText into a term list we can pass to AddVideoTranscriptModerationResult.
    List<TranscriptModerationBodyItemTermsItem> terms = new List<TranscriptModerationBodyItemTermsItem>();
    if (null != screen.Terms)
    {
        foreach (var term in screen.Terms)
        {
            if (term.Index.HasValue)
            {
                terms.Add(new TranscriptModerationBodyItemTermsItem(term.Index.Value, term.Term));
            }
        }
    }

    List<TranscriptModerationBodyItem> body = new List<TranscriptModerationBodyItem>()
    {
        new TranscriptModerationBodyItem()
        {
            Timestamp = "0",
            Terms = terms
        }
    };

    client.Reviews.AddVideoTranscriptModerationResult("application/json", TeamName, review_id, body);

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

Agregue la definición de método **Main** al espacio de nombres VideoTranscriptReviews, clase Program. Por último, cierre la clase Program y el espacio de nombres VideoTranscriptReviews.

> [!NOTE]
> El programa utiliza una transcripción de ejemplo en formato VTT. En una solución real, utilice el servicio Azure Media Indexer para [generar una transcripción](https://docs.microsoft.com/azure/media-services/media-services-index-content) a partir de un vídeo.

```csharp
static void Main(string[] args)
{
    using (ContentModeratorClient client = NewClient())
    {
        // Create a review with the content pointing to a streaming endpoint (manifest)
        var streamingcontent = "https://amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest";
        string review_id = CreateReview(client, "review1", streamingcontent);

        var transcript = @"WEBVTT

        01:01.000 --> 02:02.000
        First line with a negative word in a transcript.

        02:03.000 --> 02:25.000
        This is another line in the transcript.
        ";

        AddTranscript(client, review_id, transcript);

        AddTranscriptModerationResult(client, review_id, transcript);

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

```console
Creating a video review.
Adding a transcript to the review with ID 201801v5b08eefa0d2d4d64a1942aec7f5cacc3.
Adding a transcript moderation result to the review with ID 201801v5b08eefa0d2d4d64a1942aec7f5cacc3.
Publishing the review with ID 201801v5b08eefa0d2d4d64a1942aec7f5cacc3.
Open your Content Moderator Dashboard and select Review > Video to see the review.
Press any key to close the application.
```

## <a name="navigate-to-your-video-transcript-review"></a>Navegación a la revisión de la transcripción de vídeo

Vaya a la revisión de transcripción del vídeo en la herramienta de revisión de Content Moderator en la pantalla **Revisar**>**Vídeo**>**Transcripción**.

Verá las siguientes características:
- Las dos líneas de transcripción agregadas
- Los términos soeces encontrados y resaltados por el servicio de moderación de texto
- Al seleccionar un texto de transcripción se inicia el vídeo de esa marca de tiempo.

![Revisión de transcripción de vídeo para moderadores humanos](images/ams-video-transcript-review.PNG)

## <a name="next-steps"></a>Pasos siguientes

Obtenga el [SDK de Content Moderator para .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) y la [solución de Visual Studio](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator) para esta y otras guías de inicio rápido de Content Moderator para .NET.

Obtenga información acerca de cómo generar [revisiones de vídeo](video-reviews-quickstart-dotnet.md) en la herramienta de revisión.

Consulte el tutorial detallado sobre cómo desarrollar una [solución completa de moderación de vídeo](video-transcript-moderation-review-tutorial-dotnet.md).
