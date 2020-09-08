---
title: Uso del punto de conexión de predicción para probar imágenes mediante programación con un clasificador (Custom Vision)
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo usar la API para probar las imágenes mediante programación con el clasificador de Custom Vision Service.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 04/02/2019
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 7f1939536e033d2cf964dd2f4ee562e4ee20061b
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88934759"
---
# <a name="use-your-model-with-the-prediction-api"></a>Uso del modelo con Prediction API

Después de entrenar el modelo, puede probar imágenes mediante programación mediante su envío al punto de conexión de Prediction API.

> [!NOTE]
> En este documento se explica cómo usar C# para enviar una imagen a Prediction API. Para más información y ejemplos, consulte la [referencia de Prediction API](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.0/operations/5c82db60bf6a2b11a8247c15).

## <a name="publish-your-trained-iteration"></a>Publicación de la iteración entrenada

En la [página web de Custom Vision](https://customvision.ai), seleccione el proyecto y luego seleccione la pestaña __Rendimiento__.

Para enviar imágenes a Prediction API, primero debe publicar la iteración para predicción; para ello, puede seleccionar __Publish__ (Publicar) y especificar un nombre para la iteración publicada. Esto hará que el modelo sea accesible a la instancia de Prediction API del recurso de Azure de Custom Vision.

![Se muestra la pestaña de rendimiento, con un rectángulo rojo que rodea el botón Publish (Publicar).](./media/use-prediction-api/unpublished-iteration.png)

Una vez que el modelo se ha publicado correctamente, verá que aparece una etiqueta "Published" (Publicado) junto a la iteración en la barra lateral izquierda y que su nombre aparece en la descripción de la iteración.

![Se muestra la pestaña de rendimiento, con un rectángulo rojo que rodea la etiqueta Published (Publicado) y el nombre de la iteración publicada.](./media/use-prediction-api/published-iteration.png)

## <a name="get-the-url-and-prediction-key"></a>Obtención de una clave de predicción y dirección URL

Una vez que se ha publicado el modelo, puede recuperar la información necesaria mediante la selección de __Prediction URL__ (Dirección URL de predicción). Se abre un cuadro de diálogo con información sobre el uso de Prediction API, incluidas las opciones __Prediction URL__ (Dirección URL de predicción) y __Prediction-Key__ (Clave de predicción).

![Se muestra la pestaña de rendimiento con un rectángulo rojo que rodea el botón de dirección URL de predicción.](./media/use-prediction-api/published-iteration-prediction-url.png)

![La pestaña de rendimiento se muestra con un rectángulo rojo que rodea la dirección URL de predicción para el uso de un archivo de imagen y el valor de la clave de predicción.](./media/use-prediction-api/prediction-api-info.png)


En esta guía, se usará una imagen local, así que copie la dirección URL que aparece en **If you have an image file** (Si tiene un archivo de imagen) en una ubicación temporal. Copie también el valor correspondiente de __Prediction-Key__ (Clave de predicción).

## <a name="create-the-application"></a>Creación de la aplicación

1. En Visual Studio, cree una aplicación de consola de C#.

1. Use el siguiente código como cuerpo del archivo __Program.cs__.

    ```csharp
    using System;
    using System.IO;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Threading.Tasks;

    namespace CVSPredictionSample
    {
        public static class Program
        {
            public static void Main()
            {
                Console.Write("Enter image file path: ");
                string imageFilePath = Console.ReadLine();

                MakePredictionRequest(imageFilePath).Wait();

                Console.WriteLine("\n\nHit ENTER to exit...");
                Console.ReadLine();
            }

            public static async Task MakePredictionRequest(string imageFilePath)
            {
                var client = new HttpClient();

                // Request headers - replace this example key with your valid Prediction-Key.
                client.DefaultRequestHeaders.Add("Prediction-Key", "<Your prediction key>");

                // Prediction URL - replace this example URL with your valid Prediction URL.
                string url = "<Your prediction URL>";

                HttpResponseMessage response;

                // Request body. Try this sample with a locally stored image.
                byte[] byteData = GetImageAsByteArray(imageFilePath);

                using (var content = new ByteArrayContent(byteData))
                {
                    content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");
                    response = await client.PostAsync(url, content);
                    Console.WriteLine(await response.Content.ReadAsStringAsync());
                }
            }

            private static byte[] GetImageAsByteArray(string imageFilePath)
            {
                FileStream fileStream = new FileStream(imageFilePath, FileMode.Open, FileAccess.Read);
                BinaryReader binaryReader = new BinaryReader(fileStream);
                return binaryReader.ReadBytes((int)fileStream.Length);
            }
        }
    }
    ```

1. Cambie la siguiente información:
   * Establezca el campo `namespace` en el nombre del proyecto.
   * Reemplace el marcador de posición `<Your prediction key>` por el valor de clave que recuperó anteriormente.
   * Reemplace el marcador de posición `<Your prediction URL>` por la dirección URL que recuperó anteriormente.

## <a name="run-the-application"></a>Ejecución de la aplicación

Al ejecutar la aplicación, se le pide que escriba una ruta de acceso a un archivo de imagen de la consola. La imagen se envía luego a la instancia de Prediction API, y los resultados de predicción se devuelven como una cadena con formato JSON. Este es un ejemplo de respuesta:

```json
{
    "id":"7796df8e-acbc-45fc-90b4-1b0c81b73639",
    "project":"8622c779-471c-4b6e-842c-67a11deffd7b",
    "iteration":"59ec199d-f3fb-443a-b708-4bca79e1b7f7",
    "created":"2019-03-20T16:47:31.322Z",
    "predictions":[
        {"tagId":"d9cb3fa5-1ff3-4e98-8d47-2ef42d7fb373","tagName":"cat", "probability":1.0},
        {"tagId":"9a8d63fb-b6ed-4462-bcff-77ff72084d99","tagName":"dog", "probability":0.1087869}
    ]
}
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía, ha aprendido a enviar imágenes a su clasificador o detector de imágenes personalizado y a recibir una respuesta mediante programación con el SDK de C#. A continuación, aprenda a realizar escenarios de un extremo a otro con C# o comience a usar un SDK de un lenguaje diferente.

* [Inicio rápido: SDK de Custom Vision](quickstarts/image-classification.md)
