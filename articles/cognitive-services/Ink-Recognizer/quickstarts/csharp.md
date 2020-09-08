---
title: 'Inicio rápido: Reconocimiento de la entrada de lápiz digital con la API REST Ink Recognizer y C#'
titleSuffix: Azure Cognitive Services
description: En este inicio rápido se indica cómo usar API Ink Recognizer y C# para comenzar a reconocer los trazos de entrada de lápiz digital.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: ink-recognizer
ms.topic: quickstart
ms.date: 08/24/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: a4bb02f11c7a9a75ddc96e0ee8e9f4b868f8ade5
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051600"
---
# <a name="quickstart-recognize-digital-ink-with-the-ink-recognizer-rest-api-and-c"></a>Inicio rápido: Reconocimiento de la entrada de lápiz digital con la API REST Ink Recognizer y C#

[!INCLUDE [ink-recognizer-deprecation](../includes/deprecation-note.md)]

Use este inicio rápido para comenzar a enviar trazos de entrada de lápiz digital a la API Ink Recognizer. Esta aplicación de C# envía una solicitud de API que contiene datos del trazo de entrada de lápiz con formato JSON y muestra la respuesta.

Si bien esta aplicación está escrita en C#, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación.

Normalmente se llamaría a la API desde una aplicación de entrada de lápiz digital. En este inicio rápido se envían los datos del trazo de entrada de lápiz digital para la siguiente muestra escrita a mano desde un archivo JSON.

![imagen de un texto manuscrito](../media/handwriting-sample.jpg)

El código fuente de este inicio rápido está disponible en [GitHub](https://go.microsoft.com/fwlink/?linkid=2089502).

## <a name="prerequisites"></a>Requisitos previos

- Cualquier edición de [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/).
- [Newtonsoft.Json](https://www.newtonsoft.com/json)
    - Para instalar Newtonsoft.Json como un paquete NuGet en Visual Studio, siga estos pasos:
        1. Haga clic con el botón derecho en el **Administrador de soluciones**.
        2. Haga clic en **Administrar paquetes de NuGet...**
        3. Busque `Newtonsoft.Json` e instale el paquete.
- Si usa Linux o MacOS, esta aplicación puede ejecutarse con [Mono](https://www.mono-project.com/).

- Los datos de trazo de entrada de lápiz para este inicio rápido se pueden encontrar en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/dotnet/Vision/InkRecognition/quickstart/example-ink-strokes.json).

### <a name="create-an-ink-recognizer-resource"></a>Creación de un recurso de Ink Recognizer

[!INCLUDE [creating-an-ink-recognizer-resource](../includes/setup-instructions.md)]

## <a name="create-a-new-application"></a>Creación de una aplicación

1. En Visual Studio, cree una solución de consola y agregue los siguientes paquetes. 
    
    [!code-csharp[import statements](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=imports)]

2. Cree variables para la clave de suscripción y el punto de conexión, y el archivo JSON de ejemplo. El punto de conexión se combinará más adelante con `inkRecognitionUrl` para acceder a la API. 

    [!code-csharp[endpoint file and key variables](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=vars)]

## <a name="create-a-function-to-send-requests"></a>Creación de una función para enviar solicitudes

1. Cree una función asincrónica llamada `Request` que adopte las variables creadas anteriormente.

2. Establezca el protocolo de seguridad del cliente y la información de encabezado con un objeto `HttpClient`. No olvide agregar la clave de suscripción al encabezado `Ocp-Apim-Subscription-Key`. A continuación, cree un objeto `StringContent` para la solicitud.
 
3. Envíe la solicitud con `PutAsync()`. Si la solicitud es correcta, devuelva la respuesta.  
    
    [!code-csharp[request example method](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=request)]

## <a name="send-an-ink-recognition-request"></a>Envío de una solicitud de reconocimiento de entada de lápiz

1. Cree una nueva función llamada `recognizeInk()`. Construya la solicitud y envíela mediante una llamada a la función `Request()` con el punto de conexión, la clave de suscripción, la dirección URL de la API y los datos de trazo de entrada de lápiz digital.

2. Deserialice el objeto JSON y escríbalo en la consola. 
    
    [!code-csharp[request to recognize ink data](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=recognize)]

## <a name="load-your-digital-ink-data"></a>Carga de los datos de entrada de lápiz digital

Cree una función llamada `LoadJson()` para cargar el archivo JSON de datos de entrada de lápiz. Use `StreamReader` y `JsonTextReader` para crear un objeto `JObject` y devolverlo.

[!code-csharp[load the JSON file](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=loadJson)]

## <a name="send-the-api-request"></a>Envío de la solicitud de API

1. En el método main de la aplicación, cargue los datos JSON con la función creada anteriormente. 

2. Llame a la función `recognizeInk()` creada anteriormente. Use `System.Console.ReadKey()` para mantener abierta la ventana de consola después de ejecutar la aplicación.
    
    [!code-csharp[file main method](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=main)]


## <a name="run-the-application-and-view-the-response"></a>Ejecución de la aplicación y visualización de la respuesta

Ejecute la aplicación. Se devuelve una respuesta correcta en formato JSON. También puede encontrar la respuesta JSON en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/dotnet/Vision/InkRecognition/quickstart/example-response.json).


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Referencia de API de REST](https://go.microsoft.com/fwlink/?linkid=2089907)


Para ver el funcionamiento de la API Ink Recognition en una aplicación de entrada de lápiz digital, eche un vistazo a las siguientes aplicaciones de ejemplo en GitHub:
* [C# y Plataforma universal de Windows (UWP)](https://go.microsoft.com/fwlink/?linkid=2089803)  
* [C# y Windows Presentation Foundation (WPF)](https://go.microsoft.com/fwlink/?linkid=2089804)
* [Aplicación para explorador web de JavaScript](https://go.microsoft.com/fwlink/?linkid=2089908)       
* [Aplicación móvil Java y Android](https://go.microsoft.com/fwlink/?linkid=2089906)
* [Aplicación móvil iOS y SWIFT](https://go.microsoft.com/fwlink/?linkid=2089805)
