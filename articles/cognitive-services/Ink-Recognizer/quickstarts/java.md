---
title: 'Inicio rápido: Reconocimiento de la entrada de lápiz digital con la API REST Ink Recognizer y Java'
titleSuffix: Azure Cognitive Services
description: En este inicio rápido, usará API Ink Recognizer y Java para comenzar a reconocer los trazos de entrada de lápiz digital.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: ink-recognizer
ms.topic: quickstart
ms.date: 08/24/2020
ms.author: aahi
ms.custom: devx-track-java
ms.openlocfilehash: b73f5013fdbef34344ece672bacff90db9bac60b
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051583"
---
# <a name="quickstart-recognize-digital-ink-with-the-ink-recognizer-rest-api-and-java"></a>Inicio rápido: Reconocimiento de la entrada de lápiz digital con la API REST Ink Recognizer y Java

[!INCLUDE [ink-recognizer-deprecation](../includes/deprecation-note.md)]

Use este inicio rápido para empezar a usar la API Ink Recognizer en trazos de entrada de lápiz digital. Esta aplicación de Java envía una solicitud de API que contiene los datos del trazo de entrada de lápiz con formato JSON y obtiene la respuesta.

Si bien esta aplicación está escrita en Java, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación.

Normalmente se llamaría a la API desde una aplicación de entrada de lápiz digital. En este inicio rápido se envían los datos del trazo de entrada de lápiz digital para la siguiente muestra escrita a mano desde un archivo JSON.

![imagen de un texto manuscrito](../media/handwriting-sample.jpg)

El código fuente de este inicio rápido está disponible en [GitHub](https://go.microsoft.com/fwlink/?linkid=2089904).

## <a name="prerequisites"></a>Requisitos previos

- Kit de desarrollo de [Java&trade; (JDK) 7](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) o superior.

- Importación de estas bibliotecas desde el repositorio de Maven
    - Paquete de [JSON en Java](https://mvnrepository.com/artifact/org.json/json)
    - Paquete de [Apache HttpClient](https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient)

- Los datos de trazo de entrada de lápiz para este inicio rápido se pueden encontrar en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/java/InkRecognition/quickstart/example-ink-strokes.json).

### <a name="create-an-ink-recognizer-resource"></a>Creación de un recurso de Ink Recognizer

[!INCLUDE [creating an ink recognizer resource](../includes/setup-instructions.md)]

## <a name="create-a-new-application"></a>Creación de una aplicación

1. Cree un proyecto de Java en su IDE o editor favorito e importe las bibliotecas siguientes.
    
    [!code-java[import statements](~/cognitive-services-rest-samples/java/InkRecognition/quickstart/RecognizeInk.java?name=imports)]

2. Cree variables para la clave de suscripción y el punto de conexión, y el archivo JSON. El punto de conexión se anexará más adelante al identificador URI de Ink Recognizer.

    [!code-java[initial vars](~/cognitive-services-rest-samples/java/InkRecognition/quickstart/RecognizeInk.java?name=vars)]

## <a name="create-a-function-to-send-requests"></a>Creación de una función para enviar solicitudes

1. Cree una función llamada `sendRequest()` que adopte las variables creadas anteriormente. Después, lleve a cabo los siguiente pasos.

2. Cree un objeto `CloseableHttpClient` que pueda enviar solicitudes a la API. Envíe la solicitud a un objeto de solicitud `HttpPut` mediante la combinación del punto de conexión y la dirección URL de Ink Recognizer.

3. Use la función `setHeader()` de la solicitud para establecer el encabezado `Content-Type` en `application/json` y agregue su clave de suscripción al encabezado `Ocp-Apim-Subscription-Key`.

4. Use la función `setEntity()` de la solicitud para los datos que se van a enviar.   

5. Use la función `execute()` del cliente para enviar la solicitud y guárdela en un objeto `CloseableHttpResponse`. 

6. Cree un objeto `HttpEntity` para almacenar el contenido de respuesta. Obtenga el contenido con `getEntity()`. Si la respuesta no está vacía, se devuelve.
    
    [!code-java[send a request](~/cognitive-services-rest-samples/java/InkRecognition/quickstart/RecognizeInk.java?name=sendRequest)]

## <a name="send-an-ink-recognition-request"></a>Envío de una solicitud de reconocimiento de entada de lápiz

Cree un método llamado `recognizeInk()` para reconocer los datos de trazo de la entrada de lápiz. Llame al método `sendRequest()` creado anteriormente con el punto de conexión, dirección URL, la clave de suscripción y datos JSON. Obtenga el resultado e imprímalo en la consola.

[!code-java[recognizeInk](~/cognitive-services-rest-samples/java/InkRecognition/quickstart/RecognizeInk.java?name=recognizeInk)]

## <a name="load-your-digital-ink-data-and-send-the-request"></a>Carga de los datos de entrada de lápiz digital y envío de la solicitud

1. En el método main de la aplicación, lea el archivo JSON que contiene los datos que se agregarán a las solicitudes de lectura.

2. Llame a la función de reconocimiento de la entrada de lápiz que creó anteriormente.
    
    [!code-java[main method](~/cognitive-services-rest-samples/java/InkRecognition/quickstart/RecognizeInk.java?name=main)]


## <a name="run-the-application-and-view-the-response"></a>Ejecución de la aplicación y visualización de la respuesta

Ejecute la aplicación. Se devuelve una respuesta correcta en formato JSON. También puede encontrar la respuesta JSON en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/java/InkRecognition/quickstart/example-response.json).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Referencia de API de REST](https://go.microsoft.com/fwlink/?linkid=2089907)


Para ver el funcionamiento de la API Ink Recognition en una aplicación de entrada de lápiz digital, eche un vistazo a las siguientes aplicaciones de ejemplo en GitHub:
* [C# y Plataforma universal de Windows (UWP)](https://go.microsoft.com/fwlink/?linkid=2089803)  
* [C# y Windows Presentation Foundation (WPF)](https://go.microsoft.com/fwlink/?linkid=2089804)
* [Aplicación para explorador web de JavaScript](https://go.microsoft.com/fwlink/?linkid=2089908)       
* [Aplicación móvil Java y Android](https://go.microsoft.com/fwlink/?linkid=2089906)
* [Aplicación móvil iOS y SWIFT](https://go.microsoft.com/fwlink/?linkid=2089805)
