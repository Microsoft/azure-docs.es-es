---
title: 'Token de información predeterminado: Bing Visual Search'
titleSuffix: Azure Cognitive Services
description: Aquí se muestra cómo usar el token de información de una imagen con Bing Visual Search API para obtener información sobre esa imagen.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-visual-search
ms.topic: conceptual
ms.date: 4/26/2019
ms.author: scottwhi
ms.custom: devx-track-python, devx-track-javascript
ms.openlocfilehash: 49daf816c4075820c9efc93611f33c4854a6042d
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87849580"
---
# <a name="use-an-insights-token-to-get-insights-for-an-image"></a>Uso de un token de información para obtener los detalles de una imagen

Bing Visual Search API devuelve información sobre una imagen proporcionada. Puede proporcionar la imagen utilizando la dirección URL de la misma, un token de conclusiones, o simplemente cargando una imagen. Para obtener más información acerca de estas opciones, consulte [What is Bing Visual Search API?](overview.md) (¿Qué es Bing Visual Search API?). En este artículo se muestra cómo usar un token de información. Para ver ejemplos que demuestran cómo cargar una imagen a fin de obtener información, vea las guiás de inicio rápido siguientes:

* ([C#](quickstarts/csharp.md)

* [Java](quickstarts/java.md)

* [Node.js](quickstarts/nodejs.md)

* [Python](quickstarts/python.md)).

Si envía a Bing Visual Search una dirección URL o un token de imagen, a continuación se muestran los datos de formulario que se deben incluir en el cuerpo de POST. Los datos de formulario deben incluir el encabezado `Content-Disposition` y su parámetro `name` se debe establecer en to "knowledgeRequest". Para detalles sobre el objeto `imageInfo`, consulte la solicitud:

```json
{
    "imageInfo" : {
        "url" : "",
        "imageInsightsToken" : "",
        "cropArea" : {
            "top" : 0.1,
            "left" : 0.5,
            "right" : 0.9,
            "bottom" : 0.9
        }
    },
    "knowledgeRequest" : {
      "filters" : {
        "site" : ""
      }
    }
}
```

Los ejemplos en este artículo muestran cómo usar el token de información. Obtendrá el token de información de un objeto `Image` en una respuesta API de /images/search. Para información sobre cómo obtener el token de información, consulte [What is the Bing Image Search API?](../Bing-Image-Search/overview.md) (¿Qué es Bing Image Search API).

```
--boundary_1234-abcd
Content-Disposition: form-data; name="knowledgeRequest"

{
    "imageInfo" : {
        "imageInsightsToken" : "ccid_tmaGQ2eU*mid_D12339146CFEDF3D40...",
    }
}

--boundary_1234-abcd--
```

Para obtener ejemplos que usan el token de información, vea lo siguiente:

* [C#](#use-with-c)

* [Java](#use-with-java)

* [Node.js](#use-with-nodejs)

* [Python](#use-with-python)

## <a name="use-with-c"></a>Uso con C#

### <a name="c-prerequisites"></a>Requisitos previos de C#

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="cree un recurso de Bing Search"  target="_blank">crear un recurso de Bing Search<span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
* Cualquier versión de [Visual Studio 2019](https://www.visualstudio.com/downloads/) para obtener este código que se ejecuta en Windows.

## <a name="run-the-application"></a>Ejecución de la aplicación

Para ejecutar esta aplicación, siga estos pasos:

1. Cree una solución de consola en Visual Studio.
2. Reemplace el contenido de Program.cs por el código que se muestra en este inicio rápido.
3. Reemplace el valor `accessKey` con la clave de suscripción.
4. Reemplace el valor `insightsToken` con un token de información de una respuesta /images/search.
5. Ejecute el programa.

```csharp
using System;
using System.Text;
using System.Net;
using System.IO;
using System.Collections.Generic;
using System.Net.Http;
using System.Threading.Tasks;
using System.Threading;

namespace VisualSearchInsightsToken
{

    class Program
    {
        // Replace the accessKey string value with your valid subscription key.
        const string accessKey = "<yoursubscriptionkeygoeshere>";

        const string uriBase = "https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch";

        // Update with an insights token from an image object that the /images/search API endpoint returns.
        static string insightsToken = @"ccid_tmaGQ2eU*mid_D12339146CFEDF3D409CC7A66D2C98D0D71904D4*simid_608022145667564759*thid_OIP.tmaGQ2eUI1yq3yll!_jn9kwHaFZ";

        static string BoundaryTemplate = "batch_{0}";

        static void Main(string[] args)
        {
            try { 
                Console.WriteLine("Getting image insights using insights token: " + insightsToken);

                var knowledgeRequest = "{\"imageInfo\" : {\"imageInsightsToken\" : \"" + insightsToken + "\"}}";
                var boundary = string.Format(BoundaryTemplate, Guid.NewGuid());

                var json = BingVisualSearch(knowledgeRequest, boundary, uriBase, accessKey);

                Console.WriteLine("\nJSON Response:\n");
                Console.WriteLine(JsonPrettyPrint(json));

                Console.Write("\nPress Enter to exit ");
                Console.ReadLine();
            }
            catch (Exception e)
            {
                Console.WriteLine(e.Message);
            }

        }


        /// <summary>
        /// Calls the Bing visual search endpoint and returns the JSON response with insights.
        /// </summary>
        static string BingVisualSearch(string knowledgeRequest, string boundary, string uri, string subscriptionKey)
        {
            var requestMessage = new HttpRequestMessage(HttpMethod.Post, uri);
            requestMessage.Headers.Add("Ocp-Apim-Subscription-Key", accessKey);

            var content = new MultipartFormDataContent(boundary);
            content.Add(new StringContent(knowledgeRequest, Encoding.UTF8, "application/json"), "knowledgeRequest");
            requestMessage.Content = content;

            var httpClient = new HttpClient();

            Task<HttpResponseMessage> httpRequest = httpClient.SendAsync(requestMessage, HttpCompletionOption.ResponseContentRead, CancellationToken.None);
            HttpResponseMessage httpResponse = httpRequest.Result;
            HttpStatusCode statusCode = httpResponse.StatusCode;
            HttpContent responseContent = httpResponse.Content;

            string json = null;

            if (responseContent != null)
            {
                Task<String> stringContentsTask = responseContent.ReadAsStringAsync();
                json = stringContentsTask.Result;
            }


            return json;
        }


        /// <summary>
        /// Formats the given JSON string by adding line breaks and indents.
        /// </summary>
        /// <param name="json">The raw JSON string to format.</param>
        /// <returns>The formatted JSON string.</returns>
        static string JsonPrettyPrint(string json)
        {
            if (string.IsNullOrEmpty(json))
                return string.Empty;

            json = json.Replace(Environment.NewLine, "").Replace("\t", "");

            StringBuilder sb = new StringBuilder();
            bool quote = false;
            bool ignore = false;
            char last = ' ';
            int offset = 0;
            int indentLength = 2;

            foreach (char ch in json)
            {
                switch (ch)
                {
                    case '"':
                        if (!ignore) quote = !quote;
                        break;
                    case '\\':
                        if (quote && last != '\\') ignore = true;
                        break;
                }

                if (quote)
                {
                    sb.Append(ch);
                    if (last == '\\' && ignore) ignore = false;
                }
                else
                {
                    switch (ch)
                    {
                        case '{':
                        case '[':
                            sb.Append(ch);
                            sb.Append(Environment.NewLine);
                            sb.Append(new string(' ', ++offset * indentLength));
                            break;
                        case '}':
                        case ']':
                            sb.Append(Environment.NewLine);
                            sb.Append(new string(' ', --offset * indentLength));
                            sb.Append(ch);
                            break;
                        case ',':
                            sb.Append(ch);
                            sb.Append(Environment.NewLine);
                            sb.Append(new string(' ', offset * indentLength));
                            break;
                        case ':':
                            sb.Append(ch);
                            sb.Append(' ');
                            break;
                        default:
                            if (quote || ch != ' ') sb.Append(ch);
                            break;
                    }
                }
                last = ch;
            }

            return sb.ToString().Trim();
        }
    }
}
```

## <a name="use-with-java"></a>Uso con Java

### <a name="java-prerequisites"></a>Requisitos previos de Java

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="cree un recurso de Bing Search"  target="_blank">crear un recurso de Bing Search<span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
* [JDK 7 u 8](https://aka.ms/azure-jdks) para compilar y ejecutar este código. Puede utilizar un IDE de Java si tiene un favorito, pero bastará con un editor de texto.


## <a name="run-the-java-application"></a>Ejecución de la aplicación Java

Para ejecutar esta aplicación, siga estos pasos:

1. Descargue o instale la [biblioteca Gson de Java](https://github.com/google/gson). También se puede obtener Gson a través de Maven.
2. Cree un nuevo proyecto de Java en su IDE o editor favorito.
3. Agregue el código que se proporciona a un archivo denominado `VisualSearch.java`.
4. Reemplace el valor `subscriptionKey` con la clave de suscripción.
5. Ejecute el programa.

```java
package insightstoken;

import java.util.*;
import java.io.*;



/*
 * Gson: https://github.com/google/gson
 * Maven info:
 *     groupId: com.google.code.gson
 *     artifactId: gson
 *     version: 2.8.2
 *
 * Once you have compiled or downloaded gson-2.8.2.jar, assuming you have placed it in the
 * same folder as this file (BingImageSearch.java), you can compile and run this program at
 * the command line as follows.
 *
 * javac BingImageSearch.java -classpath .;gson-2.8.2.jar -encoding UTF-8
 * java -cp .;gson-2.8.2.jar BingImageSearch
 */

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;

// https://hc.apache.org/downloads.cgi (HttpComponents Downloads) HttpClient 4.5.5

import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.ContentType;
import org.apache.http.entity.mime.MultipartEntityBuilder;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClientBuilder;


public class InsightsToken {

    
    static String endpoint = "https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch";
    static String subscriptionKey = "<yoursubscriptionkeygoeshere>";

    // To get an insights, call the /images/search endpoint. Get the token from
    // the imageInsightsToken field in the Image object.
    static String insightsToken = "ccid_tmaGQ2eU*mid_D12339146CFEDF3D409CC7A66D2C98D0D71904D4*simid_608022145667564759*thid_OIP.tmaGQ2eUI1yq3yll!_jn9kwHaFZ";

    /**
     * @param args the command line arguments
     */
    public static void main(String[] args) {
        CloseableHttpClient httpClient = HttpClientBuilder.create().build();
        
        String knowledgeRequest = "{\"imageInfo\" : {\"imageInsightsToken\" : \"" + insightsToken + "\"}}";

        try {
            HttpEntity entity = MultipartEntityBuilder
                .create()
                .addTextBody("knowledgeRequest", knowledgeRequest, ContentType.create("application/json"))
                .build();

            HttpPost httpPost = new HttpPost(endpoint);
            httpPost.setHeader("Ocp-Apim-Subscription-Key", subscriptionKey);
            httpPost.setEntity(entity);
            HttpResponse response = httpClient.execute(httpPost);

            InputStream stream = response.getEntity().getContent();
            String json = new Scanner(stream).useDelimiter("\\A").next();

            System.out.println("\nJSON Response:\n");
            System.out.println(prettify(json));
        }
        catch (IOException e)
        {
            e.printStackTrace(System.out);
            System.exit(1);
        }
        catch (Exception e) {
            e.printStackTrace(System.out);
            System.exit(1);
        }
    }
    
    // pretty-printer for JSON; uses GSON parser to parse and re-serialize
    public static String prettify(String json_text) {
        JsonParser parser = new JsonParser();
        JsonObject json = parser.parse(json_text).getAsJsonObject();
        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        return gson.toJson(json);
    }


}
```

## <a name="use-with-nodejs"></a>Uso con Node.js

### <a name="nodejs-prerequisites"></a>Requisitos previos de Node.js

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="cree un recurso de Bing Search"  target="_blank">crear un recurso de Bing Search<span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
* Debe tener [Node.js 6](https://nodejs.org/en/download/) para ejecutar este código.

## <a name="run-the-javascript-application"></a>Ejecute la aplicación de JavaScript

Para ejecutar esta aplicación, siga estos pasos:

1. Cree una carpeta para el proyecto (o use su IDE o editor favorito).
2. Desde un terminal o símbolo del sistema, vaya a la carpeta que acaba de crear.
3. Instale los módulos de solicitud:
  
   ```
   npm install request  
   ```
1. Instale los módulos de FormData:  
   ```
   npm install form-data  
   ```
1. Cree un archivo denominado GetVisualInsights.js y agregue el siguiente código.
1. Reemplace el valor `subscriptionKey` con la clave de suscripción.
1. Ejecute el programa.  
   ```
   node GetVisualInsights.js
   ```

```javascript
var request = require('request');
var FormData = require('form-data');

var baseUri = 'https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch';
var subscriptionKey = '<yoursubscriptionkeygoeshere>';

// To get an insights, call the /images/search endpoint. Get the token from
// the imageInsightsToken field in the Image object.
var insightsToken = "ccid_tmaGQ2eU*mid_D12339146CFEDF3D409CC7A66D2C98D0D71904D4*simid_608022145667564759*thid_OIP.tmaGQ2eUI1yq3yll!_jn9kwHaFZ";

var knowledgeRequest = {
    "imageInfo" : {
        "imageInsightsToken" : insightsToken
    }
};

var form = new FormData();
form.append('knowledgeRequest', JSON.stringify(knowledgeRequest));

form.getLength(function(err, length){
  if (err) {
    return requestCallback(err);
  }

  var r = request.post(baseUri, requestCallback);
  r._form = form; 
  r.setHeader('Ocp-Apim-Subscription-Key', subscriptionKey);
});

function requestCallback(err, res, body) {
    console.log(JSON.stringify(JSON.parse(body), null, '  '))
}
```

## <a name="use-with-python"></a>Uso con Python

### <a name="python-prerequisites"></a>Requisitos previos de Python

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="cree un recurso de Bing Search"  target="_blank">crear un recurso de Bing Search<span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
* Debe tener [Python 3](https://www.python.org/) para ejecutar este código.

## <a name="run-the-python-application"></a>Ejecución de la aplicación de Python

Para ejecutar esta aplicación, siga estos pasos:

1. Cree un nuevo proyecto de Python en su IDE o editor favorito.
2. Cree un archivo denominado visualsearch.py y agregue el código que se muestra en esta guía de inicio rápido.
3. Reemplace el valor `SUBSCRIPTION_KEY` con la clave de suscripción.
4. Ejecute el programa.

```python
"""Bing Visual Search example"""

# Download and install Python at https://www.python.org/
# Run the following in a command console window
# pip3 install requests

import requests
import json

BASE_URI = 'https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch'

SUBSCRIPTION_KEY = '<yoursubscriptionkeygoeshere>'

HEADERS = {'Ocp-Apim-Subscription-Key': SUBSCRIPTION_KEY}

# To get an insights, call the /images/search endpoint. Get the token from
# the imageInsightsToken field in the Image object.
insightsToken = 'ccid_tmaGQ2eU*mid_D12339146CFEDF3D409CC7A66D2C98D0D71904D4*simid_608022145667564759*thid_OIP.tmaGQ2eUI1yq3yll!_jn9kwHaFZ'

formData = '{"imageInfo":{"imageInsightsToken":"' + insightsToken + '"}}'


file = {'knowledgeRequest': (None, formData)}


def main():

    try:
        response = requests.post(BASE_URI, headers=HEADERS, files=file)
        response.raise_for_status()
        print_json(response.json())

    except Exception as ex:
        raise ex


def print_json(obj):
    """Print the object as json"""
    print(json.dumps(obj, sort_keys=True, indent=2, separators=(',', ': ')))


# Main execution
if __name__ == '__main__':
    main()
```

## <a name="next-steps"></a>Pasos siguientes

[Creación de una aplicación web de página única de Visual Search](tutorial-bing-visual-search-single-page-app.md)  
[¿Qué es Bing Visual Search API?](overview.md)  
[Pruebe Cognitive Services](https://aka.ms/bingvisualsearchtryforfree)  
[Imágenes: Visual Search](https://aka.ms/bingvisualsearchreferencedoc)
