---
title: 'Tutorial: Generación de metadatos para imágenes de Azure'
titleSuffix: Azure Cognitive Services
description: En este tutorial, aprenderá a integrar el servicio Azure Computer Vision en una aplicación web para generar metadatos para las imágenes.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: tutorial
ms.date: 08/05/2020
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: f14bcc1b0576a5ad2f4dbbf30311f613607d3ee5
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89322337"
---
# <a name="tutorial-use-computer-vision-to-generate-image-metadata-in-azure-storage"></a>Tutorial: Uso de Computer Vision para generar metadatos de imágenes en Azure Storage

En este tutorial, aprenderá a integrar el servicio Azure Computer Vision en una aplicación web para generar metadatos para las imágenes cargadas. Esto resulta útil para los escenarios de [administración de activos digitales (DAM)](../overview.md#computer-vision-for-digital-asset-management); por ejemplo, si una empresa quiere generar rápidamente leyendas descriptivas o palabras clave de búsqueda para todas sus imágenes.

En [Laboratorio de Azure Storage y Cognitive Services](https://github.com/Microsoft/computerscience/blob/master/Labs/Azure%20Services/Azure%20Storage/Azure%20Storage%20and%20Cognitive%20Services%20(MVC).md), de GitHub, se puede encontrar una guía de aplicaciones completa y en este tutorial se trata esencialmente el ejercicio 5 de dicho laboratorio. Puede que desee crear la aplicación completa siguiendo cada paso, pero si solo desea obtener información sobre cómo integrar Computer Vision en una aplicación web existente, aquí puede leerlo.

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Crear un recurso de Computer Vision en Azure
> * Realizar análisis de imágenes en imágenes de Azure Storage
> * Adjuntar metadatos a imágenes de Azure Storage
> * Comprobar metadatos de imágenes desde el Explorador de Azure Storage

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services) antes de empezar. 

## <a name="prerequisites"></a>Prerrequisitos

- [Visual Studio 2017 Community Edition](https://www.visualstudio.com/products/visual-studio-community-vs.aspx), o cualquier versión superior, con las cargas de trabajo "Desarrollo de ASP.NET y web" y "Desarrollo de Azure" instaladas.
- Una cuenta de Azure Storage con un contenedor de blobs configurado para el almacenamiento de imágenes (siga el [ejercicio 1 del laboratorio de Azure Storage](https://github.com/Microsoft/computerscience/blob/master/Labs/Azure%20Services/Azure%20Storage/Azure%20Storage%20and%20Cognitive%20Services%20(MVC).md#Exercise1) si necesita ayuda con este paso).
- La herramienta Explorador de Azure Storage (siga el [ejercicio 2 del laboratorio de Azure Storage](https://github.com/Microsoft/computerscience/blob/master/Labs/Azure%20Services/Azure%20Storage/Azure%20Storage%20and%20Cognitive%20Services%20(MVC).md#Exercise2) si necesita ayuda con este paso).
- Una aplicación web ASP.NET con acceso a Azure Storage (siga el [ejercicio 3 del laboratorio de Azure Storage](https://github.com/Microsoft/computerscience/blob/master/Labs/Azure%20Services/Azure%20Storage/Azure%20Storage%20and%20Cognitive%20Services%20(MVC).md#Exercise3) para crear la aplicación rápidamente).

## <a name="create-a-computer-vision-resource"></a>Creación de un recurso de Computer Vision

Deberá crear un recurso de Computer Vision para su cuenta de Azure; este recurso administra el acceso al servicio Computer Vision de Azure. 

1. Siga las instrucciones que se indican en [Creación de un recurso de Azure Cognitive Services](../../cognitive-services-apis-create-account.md) para crear un recurso de Computer Vision.

1. Luego, vuelva al menú del grupo de recursos y haga clic en la suscripción de Computer Vision API que acaba de crear. Copie la dirección URL de **Punto de conexión** a cualquier lugar en que pueda recuperarla fácilmente en un momento. Luego, haga clic en **Show access keys** (Mostrar claves de acceso).

    ![Página de Azure Portal con el vínculo a las claves de acceso y a la dirección URL del punto de conexión resaltado](../Images/copy-vision-endpoint.png)
    
    [!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]


1. En la ventana siguiente, copie el valor de **KEY 1** en el Portapapeles.

    ![Cuadro de diálogo Administrar claves con el botón Copiar resaltado](../Images/copy-vision-key.png)

## <a name="add-computer-vision-credentials"></a>Adición de credenciales de Computer Vision

A continuación, agregará las credenciales necesarias a la aplicación para que pueda acceder a los recursos de Computer Vision.

Abra la aplicación web ASP.NET en Visual Studio y vaya al archivo **Web.config**, que se encuentra en la raíz del proyecto. Agregue las siguientes instrucciones a la sección `<appSettings>` del archivo, pero reemplace `VISION_KEY` por la clave que copió en el paso anterior y `VISION_ENDPOINT` por la dirección URL que guardó en el paso anterior.

```xml
<add key="SubscriptionKey" value="VISION_KEY" />
<add key="VisionEndpoint" value="VISION_ENDPOINT" />
```

Luego, en el Explorador de soluciones, haga clic con el botón derecho en el proyecto y use el comando **Administrar paquetes NuGet** para instalar el paquete **Microsoft.Azure.CognitiveServices.Vision.ComputerVision**. Dicho paquete contiene los tipos necesarios para llamar a Computer Vision API.

## <a name="add-metadata-generation-code"></a>Incorporación del código de generación de metadatos

A continuación, agregará el código que realmente saca provecho del servicio Computer Vision para crear metadatos para las imágenes. Estos pasos se aplicarán a la aplicación ASP.NET del laboratorio, pero puede adaptarlos a su propia aplicación. Lo importante es que tenga una aplicación web ASP.NET que puede cargar imágenes en un contenedor de Azure Storage, leer imágenes del mismo y mostrarlas en la vista. Si no está seguro de este paso, es mejor que siga el [ejercicio 3 del laboratorio de Azure Storage](https://github.com/Microsoft/computerscience/blob/master/Labs/Azure%20Services/Azure%20Storage/Azure%20Storage%20and%20Cognitive%20Services%20(MVC).md#Exercise3). 

1. Abra el archivo *HomeController.cs*, que encontrará en la carpeta **Controllers** del proyecto, y agregue las siguientes `using` instrucciones al principio del mismo:

    ```csharp
    using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
    using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
    ```

1. A continuación, vaya al método **Upload**; este método convierte y carga imágenes en Blob Storage. Agregue el código siguiente inmediatamente después del bloque que comienza por `// Generate a thumbnail` (o al final del proceso de creación de blobs de imágenes). Este código toma el blob que contiene la imagen (`photo`) y utiliza Computer Vision para generar una descripción de ella. Computer Vision API también genera una lista de palabras clave que se aplican a la imagen. Tanto la descripción como las palabras clave generadas se almacenan en los metadatos del blob para que se pueden recuperar más adelante.

    ```csharp
    // Submit the image to Azure's Computer Vision API
    ComputerVisionClient vision = new ComputerVisionClient(
        new ApiKeyServiceClientCredentials(ConfigurationManager.AppSettings["SubscriptionKey"]),
        new System.Net.Http.DelegatingHandler[] { });
    vision.Endpoint = ConfigurationManager.AppSettings["VisionEndpoint"];

    VisualFeatureTypes[] features = new VisualFeatureTypes[] { VisualFeatureTypes.Description };
    var result = await vision.AnalyzeImageAsync(photo.Uri.ToString(), features);

    // Record the image description and tags in blob metadata
    photo.Metadata.Add("Caption", result.Description.Captions[0].Text);

    for (int i = 0; i < result.Description.Tags.Count; i++)
    {
        string key = String.Format("Tag{0}", i);
        photo.Metadata.Add(key, result.Description.Tags[i]);
    }

    await photo.SetMetadataAsync();
    ```

1. A continuación, vaya al método **Index** en el mismo archivo. Este método enumera los blobs de imágenes almacenados en el contenedor de blobs de destino (como instancias de **IListBlobItem**) y los pasa a la vista de aplicación. Reemplace el bloque `foreach` de este método por el siguiente código. Este código llama a **CloudBlockBlob.FetchAttributes** para obtener los metadatos adjuntos de cada blob. Extrae la descripción generada por el equipo (`caption`) de los metadatos y la agrega al objeto **BlobInfo**, que se pasa a la vista.
    
    ```csharp
    foreach (IListBlobItem item in container.ListBlobs())
    {
        var blob = item as CloudBlockBlob;
    
        if (blob != null)
        {
            blob.FetchAttributes(); // Get blob metadata
            var caption = blob.Metadata.ContainsKey("Caption") ? blob.Metadata["Caption"] : blob.Name;
    
            blobs.Add(new BlobInfo()
            {
                ImageUri = blob.Uri.ToString(),
                ThumbnailUri = blob.Uri.ToString().Replace("/photos/", "/thumbnails/"),
                Caption = caption
            });
        }
    }
    ```

## <a name="test-the-app"></a>Prueba de la aplicación

Guarde los cambios en Visual Studio y presione **Ctrl+F5** para iniciar la aplicación en el explorador. Use la aplicación para cargar algunas imágenes desde la carpeta "photos" de los recursos del laboratorio o desde su propia carpeta. Cuando mueva el puntero sobre cualquiera de las imágenes en la vista, debería aparecer una ventana de información sobre herramientas debe aparecer y mostrar la leyenda que ha generado el equipo de la imagen.

![La leyenda generada por el equipo](../Images/thumbnail-with-tooltip.png)

Para ver todos los metadatos adjuntos, use el Explorador de Azure Storage para ver el contenedor de almacenamiento que usa para las imágenes. Haga clic con el botón derecho en cualquiera de los blobs del mismo y seleccione **Propiedades**. En el cuadro de diálogo, verá una lista de pares clave-valor. La descripción de la imagen generada por el equipo se almacena en el elemento "Caption" y las palabras clave de búsqueda se almacenan en "Tag0", "Tag1", y así sucesivamente. Cuando haya terminado, haga clic en **Cancelar** para cerrar el cuadro de diálogo.

![Ventana del cuadro de diálogo Propiedades de imagen en la que se muestran las etiquetas de metadatos](../Images/blob-metadata.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea seguir trabajando en la aplicación web, consulte la sección [Pasos siguientes](#next-steps). Si no planea seguir usando esta aplicación, debe eliminar todos los recursos específicos de la misma. Para ello, puede eliminar el grupo de recursos que contiene la suscripción de Azure Storage y el recurso de Computer Vision. De esta forma se quita la cuenta de almacenamiento, los blobs que se han cargado en ella y el recurso de App Service necesario para conectarse con la aplicación web ASP.NET. 

Para eliminar el grupo de recursos, abra la pestaña **Grupos de recursos** del portal, vaya al grupo de recursos que usó para el proyecto y haga clic en **Eliminar grupo de recursos** en la parte superior de la vista. Se le pedirá que escriba el nombre del grupo de recursos para confirmar que desea eliminarlo, porque una vez eliminados, los grupos de recursos no se pueden recuperar.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha configurado el servicio Computer Vision de Azure en una aplicación web existente para que se generen automáticamente leyendas y palabras clave para las imágenes de los blobs en el momento en que se cargan. Luego, consulte el ejercicio 6 del laboratorio de Azure Storage, para aprender a agregar la funcionalidad de búsqueda a una aplicación web. Así se saca provecho de las palabras clave de búsqueda que genera el servicio Computer Vision.

> [!div class="nextstepaction"]
> [Agregar un cuadro de búsqueda a la aplicación](https://github.com/Microsoft/computerscience/blob/master/Labs/Azure%20Services/Azure%20Storage/Azure%20Storage%20and%20Cognitive%20Services%20(MVC).md#Exercise6)
