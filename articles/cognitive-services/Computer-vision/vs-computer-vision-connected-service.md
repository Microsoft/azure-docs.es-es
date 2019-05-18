---
title: 'Servicios conectados de Visual Studio: Computer Vision'
titleSuffix: Azure Cognitive Services
description: Conexión a Computer Vision API desde una aplicación web de ASP.NET Core mediante la característica Servicios conectados de Visual Studio.
services: cognitive-services
author: ghogen
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: tutorial
ms.date: 05/01/2019
ms.author: ghogen
ms.custom: seodec18
ms.openlocfilehash: 148f94410f6acb421d352b68b6f1ecb305a6b16a
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65235950"
---
# <a name="use-connected-services-in-visual-studio-to-connect-to-the-computer-vision-api"></a>Uso de servicios conectados en Visual Studio para conectarse a Computer Vision API

En este artículo y sus artículos complementarios se proporcionan detalles sobre el uso de la característica Servicio conectado de Visual Studio para Computer Vision API de Cognitive Services. La funcionalidad está disponible en Visual Studio 2017 15.7 o posterior, con la extensión Cognitive Services instalada.

## <a name="prerequisites"></a>Requisitos previos

- **Una suscripción de Azure**. Si no tiene, puede registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/).
- **Visual Studio 2017 15.7** con la carga de trabajo de **Desarrollo web** instalado. [Descárguelo ahora](https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs).

[!INCLUDE [vs-install-cognitive-services-vsix](../../../includes/vs-install-cognitive-services-vsix.md)]

## <a name="add-support-to-your-project-for-cognitive-services-computer-vision-api"></a>Adición de compatibilidad al proyecto para Computer Vision API de Cognitive Services

1. Cree un proyecto web ASP.NET Core. Use la plantilla de proyecto Vacío. 

1. En el **Explorador de soluciones**, elija **Agregar** > **Servicio conectado**.
   Aparece la página del servicio conectado con los servicios que puede agregar al proyecto.

   ![Menú contextual en un proyecto de Visual Studio: Agregar > Servicio conectado](../media/vs-common/Connected-Service-Menu.PNG)

1. En el menú de servicios disponibles, elija **Cognitive Services Computer Vision API**.

   ![Se describe el menú Servicios conectados: Analizar imágenes...](./media/vs-computer-vision-connected-service/Cog-Vision-Connected-Service-0.PNG)

   Si ha iniciado sesión en Visual Studio y tiene una suscripción de Azure asociada a su cuenta, aparece una página con una lista desplegable con las suscripciones.

   ![Ventana de Computer Vision API con la lista desplegable de suscripciones resaltada](media/vs-computer-vision-connected-service/Cog-Vision-Connected-Service-1.PNG)

1. Seleccione la suscripción que desea utilizar y, a continuación, elija un nombre para Computer Vision API, o elija el vínculo Editar para modificar el nombre generado automáticamente, elija el grupo de recursos y el plan de tarifa.

   ![Edición de detalles del servicio conectado](media/vs-computer-vision-connected-service/Cog-Vision-Connected-Service-2.PNG)

   Siga el vínculo para obtener más información sobre los planes de tarifa.

1. Elija Agregar para agregar compatibilidad con el servicio conectado.
   Visual Studio modifica su proyecto para agregar paquetes NuGet, las entradas del archivo de configuración y otros cambios para admitir una conexión con Computer Vision API. La Ventana de salida muestra el registro de lo que sucede en el proyecto. Debe ver algo parecido a lo siguiente:

   ```output
   [4/26/2018 5:15:31.664 PM] Adding Computer Vision API to the project.
   [4/26/2018 5:15:32.084 PM] Creating new ComputerVision...
   [4/26/2018 5:15:32.153 PM] Creating new Resource Group...
   [4/26/2018 5:15:40.286 PM] Installing NuGet package 'Microsoft.Azure.CognitiveServices.Vision.ComputerVision' version 2.1.0.
   [4/26/2018 5:15:44.117 PM] Retrieving keys...
   [4/26/2018 5:15:45.602 PM] Changing appsettings.json setting: ComputerVisionAPI_ServiceKey=<service key>
   [4/26/2018 5:15:45.606 PM] Changing appsettings.json setting: ComputerVisionAPI_ServiceEndPoint=https://australiaeast.api.cognitive.microsoft.com/vision/v2.0
   [4/26/2018 5:15:45.609 PM] Changing appsettings.json setting: ComputerVisionAPI_Name=WebApplication-Core-ComputerVision_ComputerVisionAPI
   [4/26/2018 5:15:46.747 PM] Successfully added Computer Vision API to the project.
   ```
 
## <a name="use-the-computer-vision-api-to-detect-attributes-of-an-image"></a>Uso de Computer Vision API para detectar los atributos de una imagen

1. Agregue lo siguiente usando las instrucciones de Startup.cs.
 
   ```csharp
   using System.IO;
   using System.Text;
   using Microsoft.Extensions.Configuration;
   using System.Net.Http;
   using System.Net.Http.Headers;
   ```
 
1. Agregue un campo de configuración y después un constructor que inicializa el campo de configuración de la clase `Startup` para habilitar la configuración del programa.

   ```csharp
      private IConfiguration configuration;

      public Startup(IConfiguration configuration)
      {
          this.configuration = configuration;
      }
   ```

1. En la carpeta wwwroot del proyecto, agregue una carpeta de imágenes y agregue un archivo de imagen a la carpeta wwwroot. Por ejemplo, puede usar una de las imágenes en esta [página de Computer Vision API](https://azure.microsoft.com/services/cognitive-services/computer-vision/). Haga clic con el botón derecho en una de las imágenes, guárdela en la unidad de disco duro local y, a continuación, en el Explorador de soluciones, haga clic con el botón derecho en la carpeta imágenes y seleccione **Agregar** > **Elemento existente** para agregarlo al proyecto. El proyecto debe tener un aspecto similar al siguiente en el Explorador de soluciones: 
  
   ![Captura de pantalla de la vista del explorador de soluciones con un archivo de imagen seleccionado](media/vs-computer-vision-connected-service/Cog-Vision-Connected-Service-3.PNG) 

1. Haga clic con el botón derecho en el archivo de imagen, elija Propiedades y luego seleccione **Copiar si es posterior**. 

   ![Ventana de propiedades de imagen; Copiar en el directorio de salida establecido en Copiar si es posterior](media/vs-computer-vision-connected-service/Cog-Vision-Connected-Service-5.PNG) 
 
1. Reemplace el método de configuración con el código siguiente para acceder a Computer Vision API y probar una imagen.

   ```csharp
    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        // TODO: Change this to your image's path on your site. 
        string imagePath = @"images/subway.png";

        // Enable static files such as image files. 
        app.UseStaticFiles();

        string visionApiKey = this.configuration["ComputerVisionAPI_ServiceKey"];
        string visionApiEndPoint = this.configuration["ComputerVisionAPI_ServiceEndPoint"];

        HttpClient client = new HttpClient();

        // Request headers.
        client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", visionApiKey);

        // Request parameters. A third optional parameter is "details".
        string requestParameters = "visualFeatures=Categories,Description,Color&language=en";

        // Assemble the URI for the REST API Call.
        string uri = visionApiEndPoint + "/analyze" + "?" + requestParameters;

        HttpResponseMessage response;

        // Request body. Posts an image you've added to your site's images folder. 
        var fileInfo = env.WebRootFileProvider.GetFileInfo(imagePath);
        byte[] byteData = GetImageAsByteArray(fileInfo.PhysicalPath);

        string contentString = string.Empty;
        using (ByteArrayContent content = new ByteArrayContent(byteData))
        {
            // This example uses content type "application/octet-stream".
            // The other content types you can use are "application/json" and "multipart/form-data".
            content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");

            // Execute the REST API call.
            response = client.PostAsync(uri, content).Result;

            // Get the JSON response.
            contentString = response.Content.ReadAsStringAsync().Result;
        }

        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.Run(async (context) =>
        {
            await context.Response.WriteAsync("<h1>Cognitive Services Demo</h1>");
            await context.Response.WriteAsync($"<p><b>Test Image:</b></p>");
            await context.Response.WriteAsync($"<div><img src=\"" + imagePath + "\" /></div>");
            await context.Response.WriteAsync($"<p><b>Computer Vision API results:</b></p>");
            await context.Response.WriteAsync("<p>");
            await context.Response.WriteAsync(JsonPrettyPrint(contentString));
            await context.Response.WriteAsync("<p>");
        });
    }
   ```

    Este código construye una solicitud HTTP con el URI y la imagen como contenido binario de una llamada a Computer Vision API de REST.

1. Agregue las aplicaciones auxiliares GetImageAsByteArray y JsonPrettyPrint.

   ```csharp
    /// <summary>
    /// Returns the contents of the specified file as a byte array.
    /// </summary>
    /// <param name="imageFilePath">The image file to read.</param>
    /// <returns>The byte array of the image data.</returns>
    static byte[] GetImageAsByteArray(string imageFilePath)
    {
        FileStream fileStream = new FileStream(imageFilePath, FileMode.Open, FileAccess.Read);
        BinaryReader binaryReader = new BinaryReader(fileStream);
        return binaryReader.ReadBytes((int)fileStream.Length);
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

        string INDENT_STRING = "    ";
        var indent = 0;
        var quoted = false;
        var sb = new StringBuilder();
        for (var i = 0; i < json.Length; i++)
        {
            var ch = json[i];
            switch (ch)
            {
                case '{':
                case '[':
                    sb.Append(ch);
                    if (!quoted)
                    {
                        sb.AppendLine();
                    }
                    break;
                case '}':
                case ']':
                    if (!quoted)
                    {
                        sb.AppendLine();
                    }
                    sb.Append(ch);
                    break;
                case '"':
                    sb.Append(ch);
                    bool escaped = false;
                    var index = i;
                    while (index > 0 && json[--index] == '\\')
                        escaped = !escaped;
                    if (!escaped)
                        quoted = !quoted;
                    break;
                case ',':
                    sb.Append(ch);
                    if (!quoted)
                    {
                        sb.AppendLine();
                    }
                    break;
                case ':':
                    sb.Append(ch);
                    if (!quoted)
                        sb.Append(" ");
                    break;
                default:
                    sb.Append(ch);
                    break;
            }
        }
        return sb.ToString();
    }
   ```

1. Ejecute la aplicación web y vea qué encontró Computer Vision API en la imagen.

   ![Imagen de Computer Vision API y resultados con formato](media/vs-computer-vision-connected-service/Cog-Vision-Connected-Service-4.PNG)  

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite el grupo de recursos, elimínelo. De esta forma, se eliminan la instancia de Cognitive Services y los recursos relacionados. Para eliminar el grupo de recursos mediante el portal:

1. Escriba el nombre del grupo de recursos en el cuadro de búsqueda de la parte superior del portal. Cuando vea el grupo de recursos que se utiliza en esta guía de inicio rápido en los resultados de búsqueda, selecciónelo.
2. Seleccione **Eliminar grupo de recursos**.
3. En el cuadro **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS:** escriba el nombre del grupo de recursos y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Computer Vision API, lea la [documentación de Computer Vision API](Home.md).
