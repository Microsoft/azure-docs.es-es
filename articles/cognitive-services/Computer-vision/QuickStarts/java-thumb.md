---
title: 'Inicio rápido: Generación de una miniatura (REST, Java)'
titleSuffix: Azure Cognitive Services
description: En esta guía de inicio rápido, generará una miniatura de una imagen mediante Computer Vision API con Java.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: quickstart
ms.date: 08/05/2020
ms.author: pafarley
ms.custom: seodec18, devx-track-java
ms.openlocfilehash: 93f4e8b338f63f19a8c92bffd709a5f408072818
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87835126"
---
# <a name="quickstart-generate-a-thumbnail-using-the-computer-vision-rest-api-and-java"></a>Inicio rápido: Generación de una miniatura mediante la API REST Computer Vision y Java

En este inicio rápido, generará una miniatura a partir de una imagen mediante la API REST de Computer Vision. Debe especificar el alto y el ancho, que pueden ser diferentes a la relación de aspecto de la imagen de entrada. Computer Vision usa el recorte inteligente para identificar el área de interés de forma inteligente y generar coordenadas de recorte en función de esa región.

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* Plataforma [Java&trade;, kit de desarrollo de edición estándar 7 u 8](https://aka.ms/azure-jdks) (JDK 7 o 8)
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="Creación de un recurso de Computer Vision"  target="_blank">cree un recurso de Computer Vision <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación al servicio Computer Vision. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* [Cree las variables de entorno](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication) `COMPUTER_VISION_SUBSCRIPTION_KEY` y `COMPUTER_VISION_ENDPOINT` para la clave y la dirección URL del punto de conexión, respectivamente.

## <a name="create-and-run-the-sample-application"></a>Creación y ejecución de la aplicación de ejemplo

Para crear y ejecutar el ejemplo, siga estos pasos:

1. Cree un nuevo proyecto de Java en su IDE o editor favorito. Si la opción está disponible, cree el proyecto de Java desde una plantilla de aplicación de línea de comandos.
1. Importe las bibliotecas siguientes en el proyecto de Java. Si usa Maven, se proporcionan las coordenadas de Maven para cada biblioteca.
   - [Cliente HTTP de Apache](https://hc.apache.org/downloads.cgi) (org.apache.httpcomponents:httpclient:4.5.X)
   - [Núcleo HTTP de Apache](https://hc.apache.org/downloads.cgi) (org.apache.httpcomponents:httpcore:4.4.X)
   - [Biblioteca JSON](https://github.com/stleary/JSON-java) (org.json:json:20180130)
1. Agregue las instrucciones `import` siguientes a su clase principal: 

   ```java
    import java.awt.*;
    import javax.swing.*;
    import java.net.URI;
    import java.io.InputStream;
    import javax.imageio.ImageIO;
    import java.awt.image.BufferedImage;
    import org.apache.http.HttpEntity;
    import org.apache.http.HttpResponse;
    import org.apache.http.client.methods.HttpPost;
    import org.apache.http.entity.StringEntity;
    import org.apache.http.client.utils.URIBuilder;
    import org.apache.http.impl.client.CloseableHttpClient;
    import org.apache.http.impl.client.HttpClientBuilder;
    import org.apache.http.util.EntityUtils;
    import org.json.JSONObject;
   ```

1. Agregue el resto del código de ejemplo siguiente, debajo de las importaciones (cambie al nombre de clase si es necesario).
1. Agregue la clave de suscripción y el punto de conexión de Computer Vision a las variables de entorno.
1. Opcionalmente, reemplace el valor de `imageToAnalyze` por la dirección URL de su propia imagen.
1. Guárdela y compile el proyecto de Java.
1. Si usa un IDE, ejecute `GenerateThumbnail`. De lo contrario, ejecute desde la línea de comandos (comandos a continuación).

```java
/**
 * This sample uses the following libraries (create a "lib" folder to place them in): 
 * Apache HTTP client:
 * org.apache.httpcomponents:httpclient:4.5.X 
 * Apache HTTP core:
 * org.apache.httpcomponents:httpccore:4.4.X 
 * JSON library:
 * org.json:json:20180130
 *
 * To build/run from the command line: 
 *     javac GenerateThumbnail.java -cp .;lib\*
 *     java -cp .;lib\* GenerateThumbnail
 */

public class GenerateThumbnail {

    // Add your Computer Vision subscription key and endpoint to your environment
    // variables. Then, close and then re-open your command shell or project for the
    // changes to take effect.
    private static String subscriptionKey = System.getenv("COMPUTER_VISION_SUBSCRIPTION_KEY");
    private static String endpoint = System.getenv("COMPUTER_VISION_ENDPOINT");
    // The endpoint path
    private static final String uriBase = endpoint + "vision/v3.0/generateThumbnail";
    // It's optional if you'd like to use your own image instead of this one.
    private static final String imageToAnalyze = "https://upload.wikimedia.org/wikipedia/commons/9/94/Bloodhound_Puppy.jpg";

    public static void main(String[] args) {
        CloseableHttpClient httpClient = HttpClientBuilder.create().build();

        try {
            URIBuilder uriBuilder = new URIBuilder(uriBase);

            // Request parameters.
            uriBuilder.setParameter("width", "100");
            uriBuilder.setParameter("height", "150");
            uriBuilder.setParameter("smartCropping", "true");

            // Prepare the URI for the REST API method.
            URI uri = uriBuilder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKey);

            // Request body.
            StringEntity requestEntity = new StringEntity("{\"url\":\"" + imageToAnalyze + "\"}");
            request.setEntity(requestEntity);

            // Call the REST API method and get the response entity.
            HttpResponse response = httpClient.execute(request);
            HttpEntity entity = response.getEntity();

            System.out.println("status" + response.getStatusLine().getStatusCode());

            // Check for success.
            if (response.getStatusLine().getStatusCode() == 200) {
                // Display the thumbnail.
                System.out.println("\nDisplaying thumbnail.\n");
                displayImage(entity.getContent());
            } else {
                // Format and display the JSON error message.
                String jsonString = EntityUtils.toString(entity);
                JSONObject json = new JSONObject(jsonString);
                System.out.println("Error:\n");
                System.out.println(json.toString(2));
            }
        } catch (Exception e) {
            System.out.println(e.getMessage());
            e.printStackTrace();
        }
    }

    // Displays the given input stream as an image.
    public static void displayImage(InputStream inputStream) {
        try {
            BufferedImage bufferedImage = ImageIO.read(inputStream);

            ImageIcon imageIcon = new ImageIcon(bufferedImage);

            JLabel jLabel = new JLabel();
            jLabel.setIcon(imageIcon);

            JFrame jFrame = new JFrame();
            jFrame.setLayout(new FlowLayout());
            jFrame.setSize(100, 150);

            jFrame.add(jLabel);
            jFrame.setVisible(true);
            jFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
}
```

## <a name="examine-the-response"></a>Examen de la respuesta

Se devuelve una respuesta correcta como datos binarios, que representan los datos de imagen para la miniatura. Si la solicitud se realiza correctamente, la miniatura se genera a partir de los datos binarios de la respuesta y se muestra en una ventana independiente creada por la aplicación de ejemplo. Si se produce un error en la solicitud, la respuesta se muestra en la ventana de consola. La respuesta de la solicitud con error contiene un código de error y un mensaje para ayudar a determinar qué salió mal.

## <a name="next-steps"></a>Pasos siguientes

Explore una aplicación de Java Swing que usa Computer Vision para realizar el reconocimiento óptico de caracteres (OCR), crear miniaturas con recorte inteligente, y detectar, clasificar, etiquetar y describir características visuales en una imagen.

> [!div class="nextstepaction"]
> [Computer Vision API Java Tutorial](../Tutorials/java-tutorial.md) (Tutorial de Computer Vision API para Java)

* Para experimentar rápidamente con la versión de Computer Vision API, pruebe la [consola de pruebas de Open API](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa/console).
