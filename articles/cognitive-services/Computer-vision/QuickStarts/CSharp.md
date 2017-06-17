--- 
title: Computer Vision API C# quick starts | Microsoft Docs
description: Get information and code samples to help you quickly get started using C# and the Computer Vision API in Cognitive Services.
services: cognitive-services
author: JuliaNik
manager: ytkuo
ms.service: cognitive-services
ms.technology: computer-vision
ms.topic: article
ms.date: 02/20/2017
ms.author: juliakuz
ms.translationtype: Human Translation
ms.sourcegitcommit: ef74361c7a15b0eb7dad1f6ee03f8df707a7c05e
ms.openlocfilehash: e9cdd793e8091ff4207c61e41e4cdb67c386a68b
ms.contentlocale: es-es
ms.lasthandoff: 05/25/2017

---

# <a name="computer-vision-c-quick-starts"></a>Inicios rápidos de Computer Vision con C#

Este artículo proporciona información y ejemplos de código para ayudarle a comenzar rápidamente a utilizar la API de Computer Vision con C# y lograr las siguientes tareas:
* [Analizar una imagen](#AnalyzeImage)
* [Usar un modelo de dominio específico](#DomainSpecificModel)
* [Generar inteligentemente una miniatura](#GetThumbnail)
* [Detectar y extraer texto impreso de una imagen](#OCR)
* [Detectar y extraer texto manuscrito de una imagen](#RecognizeText)

## <a name="prerequisites"></a>Prerrequisitos

* Obtenga el SDK de Windows de la API de Microsoft Computer Vision [aquí](https://github.com/Microsoft/Cognitive-vision-windows).
* Para usar la API de Computer Vision, necesita una clave de suscripción. Puede obtener claves de suscripción gratuitas [aquí](https://docs.microsoft.com/en-us/azure/cognitive-services/Computer-vision/Vision-API-How-to-Topics/HowToSubscribe).

## Analizar una imagen usando Computer Vision API en C# <a name="AnalyzeImage"> </a>

Con el [método Analyze Image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/56f91f2d778daf23d8ec6739/operations/56f91f2e778daf14a499e1fa), puede extraer características visuales basadas en el contenido de la imagen. Puede subir una imagen o especificar una URL de imagen y elegir las características que desee recibir como resultado, por ejemplo:
* La categoría definida en esta [taxonomía](https://docs.microsoft.com/en-us/azure/cognitive-services/computer-vision/category-taxonomy).
* Una lista detallada de etiquetas relacionadas con el contenido de la imagen.
* Una descripción del contenido de la imagen en una oración completa.
* Las coordenadas, el sexo y la edad de las caras encontradas en la imagen.
* El ImageType (imágenes prediseñadas o un dibujo lineal).
* El color dominante, el *color accent*, o si una imagen está en blanco y negro.
* ¿La imagen contiene contenido para adultos o sexualmente sugestivo?

### <a name="analyze-an-image-c-example-request"></a>Ejemplo de solicitud para analizar una imagen con C#

Cree una nueva Aplicación de Consola en Visual Studio y, a continuación, reemplace Program.cs con el siguente código. Cambie el `string uri` para usar la ubicación donde obtuvo sus claves de suscripción y reemplace el valor de "Ocp-Apim-Subscription-Key" por su clave de suscripción.

```c#
using System;
using System.IO;
using System.Net.Http;
using System.Net.Http.Headers;

namespace CSHttpClientSample
{
    static class Program
    {
        static void Main()
        {
            Console.Write("Enter image file path: ");
            string imageFilePath = Console.ReadLine();

            MakeAnalysisRequest(imageFilePath);

            Console.WriteLine("\n\n\nHit ENTER to exit...");
            Console.ReadLine();
        }

        static byte[] GetImageAsByteArray(string imageFilePath)
        {
            FileStream fileStream = new FileStream(imageFilePath, FileMode.Open, FileAccess.Read);
            BinaryReader binaryReader = new BinaryReader(fileStream);
            return binaryReader.ReadBytes((int)fileStream.Length);
        }

        static async void MakeAnalysisRequest(string imageFilePath)
        {
            var client = new HttpClient();

            // Request headers. NOTE: Replace this example key with a valid subscription key.
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "13hc77781f7e4b19b5fcdd72a8df7156");

            // Request parameters. A third optional parameter is "details".
            string requestParameters = "visualFeatures=Categories&language=en";

            // NOTE: You must use the same location in your REST call as you used to obtain your subscription keys.
            //   For example, if you obtained your subscription keys from westus, replace "westcentralus" in the 
            //   URI below with "westus".
            string uri = "https://westcentralus.api.cognitive.microsoft.com/vision/v1.0/analyze?" + requestParameters;
            Console.WriteLine(uri);

            HttpResponseMessage response;

            // Request body. Try this sample with a locally stored JPEG image.
            byte[] byteData = GetImageAsByteArray(imageFilePath);

            using (var content = new ByteArrayContent(byteData))
            {
                // This example uses content type "application/octet-stream".
                // The other content types you can use are "application/json" and "multipart/form-data".
                content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");
                response = await client.PostAsync(uri, content);
            }
        }
    }
}
```
### <a name="analyze-an-image-response"></a>Ejemplo de respuesta del análisis de imagen

Una respuesta exitosa es retornada en formato JSON. A continuación se muestra un ejemplo:

```json
{
  "categories": [
    {
      "name": "abstract_",
      "score": 0.00390625
    },
    {
      "name": "people_",
      "score": 0.83984375,
      "detail": {
        "celebrities": [
          {
            "name": "Satya Nadella",
            "faceRectangle": {
              "left": 597,
              "top": 162,
              "width": 248,
              "height": 248
            },
            "confidence": 0.999028444
          }
        ]
      }
    }
  ],
  "adult": {
    "isAdultContent": false,
    "isRacyContent": false,
    "adultScore": 0.0934349000453949,
    "racyScore": 0.068613491952419281
  },
  "tags": [
    {
      "name": "person",
      "confidence": 0.98979085683822632
    },
    {
      "name": "man",
      "confidence": 0.94493889808654785
    },
    {
      "name": "outdoor",
      "confidence": 0.938492476940155
    },
    {
      "name": "window",
      "confidence": 0.89513939619064331
    }
  ],
  "description": {
    "tags": [
      "person",
      "man",
      "outdoor",
      "window",
      "glasses"
    ],
    "captions": [
      {
        "text": "Satya Nadella sitting on a bench",
        "confidence": 0.48293603002174407
      }
    ]
  },
  "requestId": "0dbec5ad-a3d3-4f7e-96b4-dfd57efe967d",
  "metadata": {
    "width": 1500,
    "height": 1000,
    "format": "Jpeg"
  },
  "faces": [
    {
      "age": 44,
      "gender": "Male",
      "faceRectangle": {
        "left": 593,
        "top": 160,
        "width": 250,
        "height": 250
      }
    }
  ],
  "color": {
    "dominantColorForeground": "Brown",
    "dominantColorBackground": "Brown",
    "dominantColors": [
      "Brown",
      "Black"
    ],
    "accentColor": "873B59",
    "isBWImg": false
  },
  "imageType": {
    "clipArtType": 0,
    "lineDrawingType": 0
  }
}

```

## Usar un modelo de dominio específico <a name="DomainSpecificModel"> </a>

El modelo de dominio específico es un modelo entrenado para identificar un conjunto específico de objetos en una imagen. Los dos modelos de dominio específico que están disponibles actualmente son las celebridades y puntos de referencia. El siguiente ejemplo identifica un punto de referencia en una imagen.

### <a name="landmark-c-example-request"></a>Ejemplo de solicitud para puntos de referencia

Cree una nueva Aplicación de Consola en Visual Studio, a continuación, reemplace Program.cs con el siguente código. Cambie el `string uri` para usar la ubicación donde obtuvo sus claves de suscripción y reemplace el valor de "Ocp-Apim-Subscription-Key" por su clave de suscripción.

```c#
using System;
using System.IO;
using System.Net.Http;
using System.Net.Http.Headers;

namespace CSHttpClientSample
{
    static class Program
    {
        static void Main()
        {
            Console.Write("Enter image file path: ");
            string imageFilePath = Console.ReadLine();

            MakeAnalysisRequest(imageFilePath);

            Console.WriteLine("\n\nHit ENTER to exit...\n");
            Console.ReadLine();
        }

        static byte[] GetImageAsByteArray(string imageFilePath)
        {
            FileStream fileStream = new FileStream(imageFilePath, FileMode.Open, FileAccess.Read);
            BinaryReader binaryReader = new BinaryReader(fileStream);
            return binaryReader.ReadBytes((int)fileStream.Length);
        }

        static async void MakeAnalysisRequest(string imageFilePath)
        {
            var client = new HttpClient();

            // Request headers. Replace the example key with a valid subscription key.
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "13hc77781f7e4b19b5fcdd72a8df7156");

            // Request parameters. Change "landmarks" to "celebrities" on requestParameters and uri to use the Celebrities model.
            string requestParameters = "model=landmarks";
            
            // NOTE: You must use the same location in your REST call as you used to obtain your subscription keys.
            //   For example, if you obtained your subscription keys from westus, replace "westcentralus" in the 
            //   URI below with "westus".
            string uri = "https://westcentralus.api.cognitive.microsoft.com/vision/v1.0/models/landmarks/analyze?" + requestParameters;
            Console.WriteLine(uri);

            HttpResponseMessage response;

            // Request body. Try this sample with a locally stored JPEG image.
            byte[] byteData = GetImageAsByteArray(imageFilePath);

            using (var content = new ByteArrayContent(byteData))
            {
                // This example uses content type "application/octet-stream".
                // The other content types you can use are "application/json" and "multipart/form-data".
                content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");
                response = await client.PostAsync(uri, content);
                string contentString = await response.Content.ReadAsStringAsync();
                Console.WriteLine("Response:\n");
                Console.WriteLine(contentString);
            }
        }
    }
}
```

### <a name="landmark-example-response"></a>Ejemplo de respuesta de punto de referencia

Una respuesta exitosa es retornada en formato JSON. A continuación se muestra un ejemplo:

```json
{
  "requestId": "b15f13a4-77d9-4fab-a701-7ad65bcdcaed",
  "metadata": {
    "width": 1024,
    "height": 680,
    "format": "Jpeg"
  },
  "result": {
    "landmarks": [
      {
        "name": "Space Needle",
        "confidence": 0.9448209
      }
    ]
  }
}
```

## Obtener una miniatura usando Computer Vision API en C# <a name="GetThumbnail"> </a>

Utilice el método [Get Thumbnail](https://westcentralus.dev.cognitive.microsoft.com/docs/services/56f91f2d778daf23d8ec6739/operations/56f91f2e778daf14a499e1fb) para recortar una imagen basada en su región de interés con la altura y el ancho que desee. Puede elegir incluso una relación de aspecto distinta a la relación de aspecto de la imagen de entrada.

### <a name="get-a-thumbnail-c-example-request"></a>Ejemplo de solicitud para obtener una miniatura

Cree una nueva Aplicación de Consola en Visual Studio, a continuación, reemplace Program.cs con el siguente código. Cambie el `string uri` para usar la ubicación donde obtuvo sus claves de suscripción y reemplace el valor de "Ocp-Apim-Subscription-Key" por su clave de suscripción.

```c#
using System;
using System.IO;
using System.Net.Http;
using System.Net.Http.Headers;

namespace CSHttpClientSample
{
    static class Program
    {
        static void Main()
        {
            Console.Write("Enter image file path: ");
            string imageFilePath = Console.ReadLine();

            MakeThumbNailRequest(imageFilePath);

            Console.WriteLine("\n\n\nHit ENTER to exit...");
            Console.ReadLine();
        }

        static byte[] GetImageAsByteArray(string imageFilePath)
        {
            FileStream fileStream = new FileStream(imageFilePath, FileMode.Open, FileAccess.Read);
            BinaryReader binaryReader = new BinaryReader(fileStream);
            return binaryReader.ReadBytes((int)fileStream.Length);
        }

        static async void MakeThumbNailRequest(string imageFilePath)
        {
            var client = new HttpClient();

            // Request headers. Replace the example key with a valid subscription key.
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "13hc77781f7e4b19b5fcdd72a8df7156");

            // Request parameters and URI.
            string requestParameters = "width=200&height=150&smartCropping=true";

            // NOTE: You must use the same location in your REST call as you used to obtain your subscription keys.
            //   For example, if you obtained your subscription keys from westus, replace "westcentralus" in the 
            //   URI below with "westus".
            string uri = "https://westcentralus.api.cognitive.microsoft.com/vision/v1.0/generateThumbnail?" + requestParameters;

            HttpResponseMessage response;

            // Request body. Try this sample with a locally stored JPEG image.
            byte[] byteData = GetImageAsByteArray(imageFilePath);

            using (var content = new ByteArrayContent(byteData))
            {
                // This example uses content type "application/octet-stream".
                // The other content types you can use are "application/json" and "multipart/form-data".
                content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");
                response = await client.PostAsync(uri, content);
            }
        }
    }
}
```
### <a name="get-a-thumbnail-response"></a>Respuesta al obtener una miniatura

Si la respuesta es exitosa, contiene el binario de imagen en miniatura.
En caso de fallo, se devuelve el código de error y un mensaje de error para ayudar a determinar lo que salió mal. El código de error podría ser uno de InvalidImageUrl, InvalidImageFormat, InvalidImageSize, InvalidThumbnailSize, NotSupportedImage, FailedToProcess, Timeout o InternalServerError.

## Reconocimiento óptico de caracteres (OCR) usando Computer Vision API en C#<a name="OCR"> </a>

Utilice  el [método Optical Character Recognition (OCR)](https://westcentralus.dev.cognitive.microsoft.com/docs/services/56f91f2d778daf23d8ec6739/operations/56f91f2e778daf14a499e1fc) para detectar el texto impreso en una imagen y extraer los caracteres reconocidos en un flujo de datos utilizables por la máquina.

### <a name="ocr-c-example-request"></a>Ejemplo de solicitud de OCR con C#

Cree una nueva Aplicación de Consola en Visual Studio, a continuación, reemplace Program.cs con el siguente código. Cambie el `string uri` para usar la ubicación donde obtuvo sus claves de suscripción y reemplace el valor de "Ocp-Apim-Subscription-Key" por su clave de suscripción.

```c#
using System;
using System.IO;
using System.Net.Http;
using System.Net.Http.Headers;

namespace CSHttpClientSample
{
    static class Program
    {
        static void Main()
        {
            Console.Write("Enter image file path: ");
            string imageFilePath = Console.ReadLine();

            MakeOCRRequest(imageFilePath);

            Console.WriteLine("\n\n\nHit ENTER to exit...");
            Console.ReadLine();
        }

        static byte[] GetImageAsByteArray(string imageFilePath)
        {
            FileStream fileStream = new FileStream(imageFilePath, FileMode.Open, FileAccess.Read);
            BinaryReader binaryReader = new BinaryReader(fileStream);
            return binaryReader.ReadBytes((int)fileStream.Length);
        }

        static async void MakeOCRRequest(string imageFilePath)
        {
            var client = new HttpClient();

            // Request headers. Replace the example key with a valid subscription key.
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "13hc77781f7e4b19b5fcdd72a8df7156");

            // Request parameters and URI
            string requestParameters = "language=unk&detectOrientation =true";
            
            // NOTE: You must use the same location in your REST call as you used to obtain your subscription keys.
            //   For example, if you obtained your subscription keys from westus, replace "westcentralus" in the 
            //   URI below with "westus".
            string uri = "https://westcentralus.api.cognitive.microsoft.com/vision/v1.0/ocr?" + requestParameters;

            HttpResponseMessage response;

            // Request body. Try this sample with a locally stored JPEG image.
            byte[] byteData = GetImageAsByteArray(imageFilePath);

            using (var content = new ByteArrayContent(byteData))
            {
                // This example uses content type "application/octet-stream".
                // The other content types you can use are "application/json" and "multipart/form-data".
                content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");
                response = await client.PostAsync(uri, content);
            }
        }
    }
}
```

### <a name="ocr-example-response"></a>Ejemplo de respuesta de OCR

En caso la respuesta sea exitosa, los resultados de OCR devueltos incluirán texto, cuadros delimitadores de regiones, líneas y palabras.
En caso de fallo, se devolverá el código de error junto con un mensaje de error. El código de error puede ser InvalidImageUrl, InvalidImageFormat, InvalidImageSize, NotSupportedImage, NotSupportedLanguage o InternalServerError.

```json
{
  "language": "en",
  "textAngle": -2.0000000000000338,
  "orientation": "Up",
  "regions": [
    {
      "boundingBox": "462,379,497,258",
      "lines": [
        {
          "boundingBox": "462,379,497,74",
          "words": [
            {
              "boundingBox": "462,379,41,73",
              "text": "A"
            },
            {
              "boundingBox": "523,379,153,73",
              "text": "GOAL"
            },
            {
              "boundingBox": "694,379,265,74",
              "text": "WITHOUT"
            }
          ]
        },
        {
          "boundingBox": "565,471,289,74",
          "words": [
            {
              "boundingBox": "565,471,41,73",
              "text": "A"
            },
            {
              "boundingBox": "626,471,150,73",
              "text": "PLAN"
            },
            {
              "boundingBox": "801,472,53,73",
              "text": "IS"
            }
          ]
        },
        {
          "boundingBox": "519,563,375,74",
          "words": [
            {
              "boundingBox": "519,563,149,74",
              "text": "JUST"
            },
            {
              "boundingBox": "683,564,41,72",
              "text": "A"
            },
            {
              "boundingBox": "741,564,153,73",
              "text": "WISH"
            }
          ]
        }
      ]
    }
  ]
}
```

## Reconocimiento de texto manuscrito usando Computer Vision API en C# <a name="RecognizeText"> </a>

Utilice el [método Recognize Handwritten Text](https://westus.dev.cognitive.microsoft.com/docs/services/56f91f2d778daf23d8ec6739/operations/587f2c6a154055056008f200) para detectar texto manuscrito en una imagen y extraer los caracteres reconocidos en un flujo de datos usable por la máquina.

### <a name="handwriting-recognition-c-example"></a>Ejemplo de reconocimiento de manuscritos con C#

Cree una nueva Aplicación de Consola en Visual Studio, a continuación, reemplace Program.cs con el siguente código. Cambie el `string uri` para usar la ubicación donde obtuvo sus claves de suscripción y reemplace el valor de "Ocp-Apim-Subscription-Key" por su clave de suscripción.

```c#
using System;
using System.IO;
using System.Collections;
using System.Collections.Generic;
using System.Net.Http;
using System.Net.Http.Headers;

namespace CSHttpClientSample
{
    static class Program
    {
        static void Main()
        {
            Console.Write("Enter image file path: ");
            string imageFilePath = Console.ReadLine();

            ReadHandwrittenText(imageFilePath);

            Console.WriteLine("\n\n\nHit ENTER to exit...");
            Console.ReadLine();
        }

        static byte[] GetImageAsByteArray(string imageFilePath)
        {
            FileStream fileStream = new FileStream(imageFilePath, FileMode.Open, FileAccess.Read);
            BinaryReader binaryReader = new BinaryReader(fileStream);
            return binaryReader.ReadBytes((int)fileStream.Length);
        }

        static async void ReadHandwrittenText(string imageFilePath)
        {
            var client = new HttpClient();

            // Request headers. Replace the example key with a valid subscription key.
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "13hc77781f7e4b19b5fcdd72a8df7156");

            // Request parameters and URI. Set "handwriting" to false for printed text.
            string requestParameter = "handwriting=true";

            // NOTE: You must use the same location in your REST call as you used to obtain your subscription keys.
            //   For example, if you obtained your subscription keys from westus, replace "westcentralus" in the 
            //   URI below with "westus".
            string uri = "https://westcentralus.api.cognitive.microsoft.com/vision/v1.0/recognizeText?" + requestParameter;

            HttpResponseMessage response = null;
            IEnumerable<string> responseValues = null;
            string operationLocation = null;

            // Request body. Try this sample with a locally stored JPEG image.
            byte[] byteData = GetImageAsByteArray(imageFilePath);
            var content = new ByteArrayContent(byteData);

            // This example uses content type "application/octet-stream".
            // You can also use "application/json" and specify an image URL.
            content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");

            try {
                response = await client.PostAsync(uri, content);
                responseValues = response.Headers.GetValues("Operation-Location");
            }
            catch (Exception e)
            {
                Console.WriteLine(e.Message);
            }

            foreach (var value in responseValues)
            {
                // This value is the URI where you can get the text recognition operation result.
                operationLocation = value;
                Console.WriteLine(operationLocation);
                break;
            }

            try
            {
                // Note: The response may not be immediately available. Handwriting recognition is an
                // async operation that can take a variable amount of time depending on the length
                // of the text you want to recognize. You may need to wait or retry this operation.
                response = await client.GetAsync(operationLocation);

                // And now you can see the response in in JSON:
                Console.WriteLine(await response.Content.ReadAsStringAsync());
            }
            catch (Exception e)
            {
                Console.WriteLine(e.Message);
            }
        }
    }
}
```

