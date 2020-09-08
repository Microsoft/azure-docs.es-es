---
title: 'Sample: Exploración de una aplicación de procesamiento de imágenes en C#'
titleSuffix: Azure Cognitive Services
description: Explore una aplicación básica de Windows que usa Computer Vision API de Azure Cognitive Services. Realice OCR, cree miniaturas y trabaje con características visuales en una imagen.
services: cognitive-services
author: PatrickFarley
manager: nolachar
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: sample
ms.date: 04/17/2019
ms.author: pafarley
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 522cf801e7c1bfdd3ed1f452c123a2db701c0c42
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936357"
---
# <a name="sample-explore-an-image-processing-app-with-c"></a>Sample: Exploración de una aplicación de procesamiento de imágenes con C#

Explore una aplicación Windows básica que utiliza Computer Vision para realizar el reconocimiento óptico de caracteres (OCR), crear miniaturas de recorte inteligente, además de detectar, clasificar, etiquetar y describir las características visuales, incluidas las caras, en una imagen. El ejemplo siguiente le permite enviar la dirección URL de una imagen o un archivo almacenado localmente. Puede usar este ejemplo de código abierto como plantilla para crear su propia aplicación para Windows con Computer Vision API y Windows Presentation Foundation (WPF), una parte de .NET Framework.

> [!div class="checklist"]
> * Obtener la aplicación de ejemplo de GitHub
> * Abrir y compilar la aplicación de ejemplo en Visual Studio
> * Ejecutar la aplicación de ejemplo e interactuar con ella para desarrollar diversos escenarios
> * Explorar los distintos escenarios incluidos con la aplicación de ejemplo

## <a name="prerequisites"></a>Requisitos previos

Antes de explorar la aplicación de ejemplo, asegúrese de haber cumplido los requisitos previos siguientes:

* Debe tener [Visual Studio 2015](https://visualstudio.microsoft.com/downloads/) o posterior.
* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="Creación de un recurso de Computer Vision"  target="_blank">cree un recurso de Computer Vision <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación al servicio Computer Vision. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="get-the-sample-app"></a>Obtención de la aplicación de ejemplo

La aplicación de ejemplo de Computer Vision está disponible en GitHub desde el repositorio `Microsoft/Cognitive-Vision-Windows`. Este repositorio también incluye el repositorio `Microsoft/Cognitive-Common-Windows` como submódulo de GIT. También puede clonar de forma recursiva este repositorio, incluido el submódulo, ya sea mediante el comando `git clone --recurse-submodules` desde la línea de comandos, o mediante el escritorio de GitHub.

Por ejemplo, para clonar de forma recursiva el repositorio para la aplicación de ejemplo de Computer Vision desde un símbolo del sistema, ejecute el siguiente comando:

```Console
git clone --recurse-submodules https://github.com/Microsoft/Cognitive-Vision-Windows.git
```

> [!IMPORTANT]
> No descargue este repositorio como un archivo ZIP. GIT no incluye submódulos cuando descarga un repositorio como un archivo ZIP.

### <a name="get-optional-sample-images"></a>Obtener imágenes de ejemplo opcionales

Si quiere, puede usar las imágenes de ejemplo incluidas con la aplicación de ejemplo de [Face](../../Face/Overview.md), disponible en GitHub desde el repositorio `Microsoft/Cognitive-Face-Windows`. Esa aplicación de ejemplo incluye una carpeta, `/Data`, que contiene varias imágenes de personas. Así mismo, puede clonar de forma recursiva este repositorio mediante los métodos descritos para la aplicación de ejemplo de Computer Vision.

Por ejemplo, para clonar de forma recursiva el repositorio para la aplicación de ejemplo de Face desde un símbolo del sistema, ejecute el siguiente comando:

```Console
git clone --recurse-submodules https://github.com/Microsoft/Cognitive-Face-Windows.git
```

## <a name="open-and-build-the-sample-app-in-visual-studio"></a>Abrir y compilar la aplicación de ejemplo en Visual Studio

En primer lugar, debe compilar la aplicación de ejemplo para que Visual Studio pueda resolver las dependencias, antes de ejecutar o explorar la aplicación de ejemplo. Para abrir y compilar la aplicación de ejemplo, realice los pasos siguientes:

1. Abra el archivo de solución de Visual Studio, `/Sample-WPF/VisionAPI-WPF-Samples.sln`, en Visual Studio.
1. Asegúrese de que la solución de Visual Studio contenga dos proyectos:  

   * SampleUserControlLibrary
   * VisionAPI-WPF-Samples  

   Si no está disponible el proyecto SampleUserControlLibrary, confirme que ha clonado de forma recursiva el repositorio `Microsoft/Cognitive-Vision-Windows`.
1. En Visual Studio, presione Control + Mayús + B o elija la opción **Compilar** en el menú de la cinta de opciones y, a continuación, elija **Compilar solución** para compilar la solución.

## <a name="run-and-interact-with-the-sample-app"></a>Ejecutar e interactuar con la aplicación de ejemplo

Puede ejecutar la aplicación de ejemplo para ver cómo interactúa con usted y con la biblioteca cliente de Computer Vision al realizar varias tareas, como generar miniaturas o etiquetar imágenes. Para ejecutar e interactuar con la aplicación de ejemplo, realice los pasos siguientes:

1. Una vez completada la compilación, presione **F5** o elija **Depurar** en el menú de la cinta de opciones y, a continuación, elija **Iniciar depuración** para ejecutar la aplicación de ejemplo.
1. Cuando se muestre la aplicación de ejemplo, elija **Subscription Key Management** (Administración de claves de suscripción) en el panel de navegación para mostrar la página Subscription Key Management (Administración de claves de suscripción).
   ![Página Subscription Key Management](../Images/Vision_UI_Subscription.PNG) (Administración de claves de suscripción)  
1. Escriba la clave de suscripción en **Clave de suscripción**.
1. Escriba la dirección URL del punto de conexión en **Punto de conexión**.  
   [!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]
1. Si no quiere escribir la clave de suscripción y la URL del punto de conexión la próxima vez que ejecute la aplicación de ejemplo, elija **Guardar configuración** para guardar la clave de suscripción y la URL del punto de conexión en el equipo. Si quiere eliminar la dirección URL del punto de conexión y la clave de suscripción que guardó previamente, elija **Eliminar configuración**.

   > [!NOTE]
   > La aplicación de ejemplo usa el almacenamiento aislado y `System.IO.IsolatedStorage` para almacenar la dirección URL del punto de conexión y la clave de suscripción.

1. En la opción **Seleccione un escenario** del panel de navegación, seleccione uno de los escenarios que actualmente se incluyen con la aplicación de ejemplo:  

   | Escenario | Descripción |
   |----------|-------------|
   |Analyze Image | Usa la operación [Analyze Image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa) para analizar una imagen local o remota. Puede elegir las características visuales y el idioma para el análisis y ver la imagen y los resultados.  |
   |Analyze Image with Domain Model | Usa la operación [List Domain Specific Models](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fd) para enumerar los modelos de dominio entre los que puede elegir, y la operación [Recognize Domain Specific Content](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e200) para analizar una imagen local o remota mediante el modelo de dominio seleccionado. También puede elegir el idioma para el análisis. |
   |Describe Image | Usa la operación [Describe Image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fe) para crear una descripción legible de una imagen local o remota. También puede elegir el idioma para la descripción. |
   |Generate Tags | Usa la operación [Tag Image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1ff) para etiquetar características visuales de una imagen local o remota. También puede elegir el idioma para las etiquetas. |
   |Recognize Text (OCR) | Usa la operación [OCR](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fc) para reconocer y extraer texto impreso de una imagen. Puede elegir el idioma que se usará o permitir que Computer Vision lo detecte automáticamente. |
   |Recognize Text V2 (English) | Usa las operaciones [Recognize Text](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/587f2c6a154055056008f200) y [Get Recognize Text Operation Result](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/587f2cf1154055056008f201) operaciones para reconocer y extraer texto manuscrito o impreso de una imagen de forma asincrónica. |
   |Get Thumbnail | Usa la operación [Get Thumbnail](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fb) para generar una miniatura de una imagen local o remota. |

   En la captura de pantalla siguiente se muestra la página proporcionada para el escenario Analyze Image, después de analizar una imagen de ejemplo.
   ![Captura de pantalla de la página Analizar imagen](../Images/Analyze_Image_Example.PNG)

## <a name="explore-the-sample-app"></a>Explorar la aplicación de ejemplo

La solución de Visual Studio para la aplicación de ejemplo de Computer Vision contiene dos proyectos:

* SampleUserControlLibrary  
  El proyecto SampleUserControlLibrary proporciona funcionalidades compartidas por varios ejemplos de Cognitive Services. El proyecto contiene lo siguiente:
  * SampleScenarios  
    Control de usuario que proporciona una presentación estandarizada, como la barra de título, el panel de navegación y el panel de contenido, para obtener ejemplos. La aplicación de ejemplo de Computer Vision usa este control en la ventana de MainWindow.xaml para mostrar las páginas del escenario y acceder a la información que se comparte entre los escenarios, como la dirección URL del punto de conexión y la clave de suscripción.
  * SubscriptionKeyPage  
    Página que proporciona un diseño estandarizado para escribir una clave de suscripción y una dirección URL de punto de conexión para la aplicación de ejemplo. La aplicación de ejemplo de Computer Vision usa esta página para administrar la clave de suscripción y la dirección URL de punto de conexión que se usan en las páginas del escenario.
  * VideoResultControl  
    Control de usuario que proporciona una presentación estandarizada de la información de vídeo. La aplicación de ejemplo de Computer Vision no usa este control.
* VisionAPI-WPF-Samples  
  Proyecto principal de la aplicación de ejemplo de Computer Vision que contiene todas las funcionalidades interesantes para Computer Vision. El proyecto contiene lo siguiente:
  * AnalyzeInDomainPage.xaml  
    Página del escenario Analyze Image con el escenario de modelo de dominio.
  * AnalyzeImage.xaml  
    Página del escenario Analyze Image.
  * DescribePage.xaml  
    Página del escenario Describe Image.
  * ImageScenarioPage.cs  
    Clase ImageScenarioPage, de la que se derivan todas las páginas de escenario de la aplicación de ejemplo. Esta clase administra funcionalidades, como el suministro de credenciales y el formato de salida, que se comparten en todas las páginas de escenarios.
  * MainWindow.xaml  
    Ventana principal de la aplicación de ejemplo. Utiliza el control SampleScenarios para presentar las páginas SubscriptionKeyPage y de escenarios.
  * OCRPage.xaml  
    Página del escenario Recognize Text (OCR).
  * RecognizeLanguage.cs  
    Clase RecognizeLanguage, que proporciona información acerca de los idiomas que admiten los distintos métodos de la aplicación de ejemplo.
  * TagsPage.xaml  
    Página del escenario Generate Tags.
  * TextRecognitionPage.xaml  
    Página del escenario Recognize Text V2 (OCR).
  * ThumbnailPage.xaml  
    Página del escenario Get Thumbnail.

### <a name="explore-the-sample-code"></a>Exploración del código de ejemplo

Las partes importantes del código de ejemplo se enmarcan con los bloques de comentarios que comienzan por `KEY SAMPLE CODE STARTS HERE` y terminan por `KEY SAMPLE CODE ENDS HERE`, para que pueda explorar la aplicación de ejemplo con mayor facilidad. Estas partes importantes del código de ejemplo contienen el código pertinente para aprender a realizar diversas tareas con la biblioteca cliente de Computer Vision API. Puede buscar `KEY SAMPLE CODE STARTS HERE` en Visual Studio para moverse entre las secciones de código de la aplicación de ejemplo de Computer Vision más pertinentes. 

Por ejemplo, en el método `UploadAndAnalyzeImageAsync`, que aparece a continuación y se incluye en AnalyzePage.xaml, se muestra cómo usar la biblioteca cliente para analizar una imagen local al invocar el método `ComputerVisionClient.AnalyzeImageInStreamAsync`.

```csharp
private async Task<ImageAnalysis> UploadAndAnalyzeImageAsync(string imageFilePath)
{
    // -----------------------------------------------------------------------
    // KEY SAMPLE CODE STARTS HERE
    // -----------------------------------------------------------------------

    //
    // Create Cognitive Services Vision API Service client.
    //
    using (var client = new ComputerVisionClient(Credentials) { Endpoint = Endpoint })
    {
        Log("ComputerVisionClient is created");

        using (Stream imageFileStream = File.OpenRead(imageFilePath))
        {
            //
            // Analyze the image for all visual features.
            //
            Log("Calling ComputerVisionClient.AnalyzeImageInStreamAsync()...");
            VisualFeatureTypes[] visualFeatures = GetSelectedVisualFeatures();
            string language = (_language.SelectedItem as RecognizeLanguage).ShortCode;
            ImageAnalysis analysisResult = await client.AnalyzeImageInStreamAsync(imageFileStream, visualFeatures, null, language);
            return analysisResult;
        }
    }

    // -----------------------------------------------------------------------
    // KEY SAMPLE CODE ENDS HERE
    // -----------------------------------------------------------------------
}
```

### <a name="explore-the-client-library"></a>Explorar la biblioteca cliente

Esta aplicación de ejemplo usa la biblioteca cliente de Computer Vision API, un contenedor cliente de C# fino para Computer Vision API en Azure Cognitive Services. La biblioteca cliente está disponible en NuGet en el paquete [Microsoft.Azure.CognitiveServices.Vision.ComputerVision](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.ComputerVision/). Cuando compiló la aplicación de Visual Studio, recuperó la biblioteca cliente desde el paquete NuGet correspondiente. También puede ver el código fuente de la biblioteca cliente en la carpeta `/ClientLibrary` del repositorio `Microsoft/Cognitive-Vision-Windows`.

La funcionalidad de la biblioteca cliente se centra en la clase `ComputerVisionClient`, en el espacio de nombres `Microsoft.Azure.CognitiveServices.Vision.ComputerVision`, mientras que los modelos usados por la clase `ComputerVisionClient` al interactuar con Computer Vision se encuentran en el espacio de nombres `Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models`. En las distintas páginas de escenarios XAML incluidas con la aplicación de ejemplo, encontrará las siguientes directivas `using` para esos espacios de nombres:

```csharp
// -----------------------------------------------------------------------
// KEY SAMPLE CODE STARTS HERE
// Use the following namespace for ComputerVisionClient.
// -----------------------------------------------------------------------
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
// -----------------------------------------------------------------------
// KEY SAMPLE CODE ENDS HERE
// -----------------------------------------------------------------------
```

Obtendrá más información sobre los distintos métodos que se incluyen con la clase `ComputerVisionClient` a medida que explore los escenarios incluidos con la aplicación de ejemplo de Computer Vision.

## <a name="explore-the-analyze-image-scenario"></a>Explorar el escenario Analyze Image

La página AnalyzePage.xaml administra este escenario. Puede elegir las características visuales y el idioma para el análisis y ver la imagen y los resultados. Para ello, la página del escenario usa uno de los métodos siguientes, en función del origen de la imagen:

* UploadAndAnalyzeImageAsync  
  Este método se usa para las imágenes locales, en las que la imagen se debe codificar como `Stream` y enviar a Computer Vision mediante una llamada al método `ComputerVisionClient.AnalyzeImageInStreamAsync`.
* AnalyzeUrlAsync  
  Este método se usa para las imágenes remotas, en las que la URL de la imagen se envía a Computer Vision mediante una llamada al método `ComputerVisionClient.AnalyzeImageAsync`.

El método `UploadAndAnalyzeImageAsync` crea una nueva instancia de `ComputerVisionClient` mediante la dirección URL del punto de conexión y la clave de suscripción especificadas. Dado que la aplicación de ejemplo analiza una imagen local, debe enviar el contenido de esa imagen a Computer Vision. Abre el archivo local especificado en `imageFilePath` para la lectura como `Stream` y, a continuación, obtiene las características visuales y el idioma seleccionado en la página del escenario. Realiza una llamada al método `ComputerVisionClient.AnalyzeImageInStreamAsync`, en que transfiere `Stream` para el archivo, las características visuales y el idioma y, a continuación, devuelve el resultado como una instancia `ImageAnalysis`. Los métodos heredados de la clase `ImageScenarioPage` presentan los resultados devueltos en la página del escenario.

El método `AnalyzeUrlAsync` crea una nueva instancia de `ComputerVisionClient` mediante la dirección URL del punto de conexión y la clave de suscripción especificadas. Obtiene las características visuales y el idioma seleccionado en la página del escenario. Realiza una llamada al método `ComputerVisionClient.AnalyzeImageInStreamAsync`, en que transfiere la URL de imagen, las características visuales y el idioma y, a continuación, devuelve el resultado como una instancia `ImageAnalysis`. Los métodos heredados de la clase `ImageScenarioPage` presentan los resultados devueltos en la página del escenario.

## <a name="explore-the-analyze-image-with-domain-model-scenario"></a>Explorar Analyze Image con el escenario de modelo de dominio.

La página AnalyzeInDomainPage.xaml administra este escenario. Puede elegir un modelo de dominio, como `celebrities` o `landmarks`, y el idioma para realizar un análisis específico del dominio de la imagen y ver la imagen y los resultados. La página del escenario usa los métodos siguientes, en función del origen de la imagen:

* GetAvailableDomainModelsAsync  
  Este método obtiene la lista de modelos de dominio disponibles de Computer Vision y rellena el control ComboBox `_domainModelComboBox` en la página, mediante el método `ComputerVisionClient.ListModelsAsync`.
* UploadAndAnalyzeInDomainImageAsync  
  Este método se usa para las imágenes locales, en las que la imagen se debe codificar como `Stream` y enviar a Computer Vision mediante una llamada al método `ComputerVisionClient.AnalyzeImageByDomainInStreamAsync`.
* AnalyzeInDomainUrlAsync  
  Este método se usa para las imágenes remotas, en las que la URL de la imagen se envía a Computer Vision mediante una llamada al método `ComputerVisionClient.AnalyzeImageByDomainAsync`.

El método `UploadAndAnalyzeInDomainImageAsync` crea una nueva instancia de `ComputerVisionClient` mediante la dirección URL del punto de conexión y la clave de suscripción especificadas. Dado que la aplicación de ejemplo analiza una imagen local, debe enviar el contenido de esa imagen a Computer Vision. Abre el archivo local especificado en `imageFilePath` para la lectura como `Stream` y, a continuación, obtiene el idioma seleccionado en la página del escenario. Realiza una llamada al método `ComputerVisionClient.AnalyzeImageByDomainInStreamAsync`, en que transfiere `Stream` para el archivo, el nombre del modelo de dominio y el idioma y, a continuación, devuelve el resultado como una instancia `DomainModelResults`. Los métodos heredados de la clase `ImageScenarioPage` presentan los resultados devueltos en la página del escenario.

El método `AnalyzeInDomainUrlAsync` crea una nueva instancia de `ComputerVisionClient` mediante la dirección URL del punto de conexión y la clave de suscripción especificadas. Obtiene el idioma seleccionado en la página del escenario. Realiza una llamada al método `ComputerVisionClient.AnalyzeImageByDomainAsync`, en que transfiere la URL de imagen, las características visuales y el idioma y, a continuación, devuelve el resultado como una instancia `DomainModelResults`. Los métodos heredados de la clase `ImageScenarioPage` presentan los resultados devueltos en la página del escenario.

## <a name="explore-the-describe-image-scenario"></a>Explorar el escenario Describe Image

La página DescribePage.xaml administra este escenario. Puede elegir un idioma para crear una descripción inteligible de la imagen y ver la imagen y los resultados. La página del escenario usa los métodos siguientes, en función del origen de la imagen:

* UploadAndDescribeImageAsync  
  Este método se usa para las imágenes locales, en las que la imagen se debe codificar como `Stream` y enviar a Computer Vision mediante una llamada al método `ComputerVisionClient.DescribeImageInStreamAsync`.
* DescribeUrlAsync  
  Este método se usa para las imágenes remotas, en las que la URL de la imagen se envía a Computer Vision mediante una llamada al método `ComputerVisionClient.DescribeImageAsync`.

El método `UploadAndDescribeImageAsync` crea una nueva instancia de `ComputerVisionClient` mediante la dirección URL del punto de conexión y la clave de suscripción especificadas. Dado que la aplicación de ejemplo analiza una imagen local, debe enviar el contenido de esa imagen a Computer Vision. Abre el archivo local especificado en `imageFilePath` para la lectura como `Stream` y, a continuación, obtiene el idioma seleccionado en la página del escenario. Realiza una llamada al método `ComputerVisionClient.DescribeImageInStreamAsync`, en que transfiere `Stream` para el archivo, el número máximo de candidatos (en este caso, 3) y el idioma y, a continuación, devuelve el resultado como una instancia `ImageDescription`. Los métodos heredados de la clase `ImageScenarioPage` presentan los resultados devueltos en la página del escenario.

El método `DescribeUrlAsync` crea una nueva instancia de `ComputerVisionClient` mediante la dirección URL del punto de conexión y la clave de suscripción especificadas. Obtiene el idioma seleccionado en la página del escenario. Realiza una llamada al método `ComputerVisionClient.DescribeImageAsync`, en que transfiere la URL de la imagen, el número máximo de candidatos (en este caso, 3) y el idioma y, a continuación, devuelve el resultado como una instancia `ImageDescription`. Los métodos heredados de la clase `ImageScenarioPage` presentan los resultados devueltos en la página del escenario.

## <a name="explore-the-generate-tags-scenario"></a>Explorar el escenario Generate Tags

La página TagsPage.xaml administra este escenario. Puede elegir un idioma para etiquetar las características visuales de una imagen y ver la imagen y los resultados. La página del escenario usa los métodos siguientes, en función del origen de la imagen:

* UploadAndGetTagsForImageAsync  
  Este método se usa para las imágenes locales, en las que la imagen se debe codificar como `Stream` y enviar a Computer Vision mediante una llamada al método `ComputerVisionClient.TagImageInStreamAsync`.
* GenerateTagsForUrlAsync  
  Este método se usa para las imágenes remotas, en las que la URL de la imagen se envía a Computer Vision mediante una llamada al método `ComputerVisionClient.TagImageAsync`.

El método `UploadAndGetTagsForImageAsync` crea una nueva instancia de `ComputerVisionClient` mediante la dirección URL del punto de conexión y la clave de suscripción especificadas. Dado que la aplicación de ejemplo analiza una imagen local, debe enviar el contenido de esa imagen a Computer Vision. Abre el archivo local especificado en `imageFilePath` para la lectura como `Stream` y, a continuación, obtiene el idioma seleccionado en la página del escenario. Realiza una llamada al método `ComputerVisionClient.TagImageInStreamAsync`, en que transfiere `Stream` para el archivo y el idioma y, a continuación, devuelve el resultado como una instancia `TagResult`. Los métodos heredados de la clase `ImageScenarioPage` presentan los resultados devueltos en la página del escenario.

El método `GenerateTagsForUrlAsync` crea una nueva instancia de `ComputerVisionClient` mediante la dirección URL del punto de conexión y la clave de suscripción especificadas. Obtiene el idioma seleccionado en la página del escenario. Realiza una llamada al método `ComputerVisionClient.TagImageAsync`, en que transfiere la URL de la imagen y el idioma y, a continuación, devuelve el resultado como una instancia `TagResult`. Los métodos heredados de la clase `ImageScenarioPage` presentan los resultados devueltos en la página del escenario.

## <a name="explore-the-recognize-text-ocr-scenario"></a>Explorar el escenario Recognize Text (OCR)

La página OCRPage.xaml administra este escenario. Puede elegir un idioma para reconocer y extraer texto impreso de una imagen y ver la imagen y los resultados. La página del escenario usa los métodos siguientes, en función del origen de la imagen:

* UploadAndRecognizeImageAsync  
  Este método se usa para las imágenes locales, en las que la imagen se debe codificar como `Stream` y enviar a Computer Vision mediante una llamada al método `ComputerVisionClient.RecognizePrintedTextInStreamAsync`.
* RecognizeUrlAsync  
  Este método se usa para las imágenes remotas, en las que la URL de la imagen se envía a Computer Vision mediante una llamada al método `ComputerVisionClient.RecognizePrintedTextAsync`.

El método `UploadAndRecognizeImageAsync` crea una nueva instancia de `ComputerVisionClient` mediante la dirección URL del punto de conexión y la clave de suscripción especificadas. Dado que la aplicación de ejemplo analiza una imagen local, debe enviar el contenido de esa imagen a Computer Vision. Abre el archivo local especificado en `imageFilePath` para la lectura como `Stream` y, a continuación, obtiene el idioma seleccionado en la página del escenario. Realiza una llamada al método `ComputerVisionClient.RecognizePrintedTextInStreamAsync`, en que indica que la orientación no se detectó y transfiere `Stream` para el archivo y el idioma y, a continuación, devuelve el resultado como una instancia `OcrResult`. Los métodos heredados de la clase `ImageScenarioPage` presentan los resultados devueltos en la página del escenario.

El método `RecognizeUrlAsync` crea una nueva instancia de `ComputerVisionClient` mediante la dirección URL del punto de conexión y la clave de suscripción especificadas. Obtiene el idioma seleccionado en la página del escenario. Realiza una llamada al método `ComputerVisionClient.RecognizePrintedTextAsync`, en que indica que la orientación no se detectó y transfiere la URL de la imagen y el idioma y, a continuación, devuelve el resultado como una instancia `OcrResult`. Los métodos heredados de la clase `ImageScenarioPage` presentan los resultados devueltos en la página del escenario.

## <a name="explore-the-recognize-text-v2-english-scenario"></a>Explorar el escenario Recognize Text V2 (English)

La página TextRecognitionPage.xaml administra este escenario. Puede elegir el modo de reconocimiento y un idioma para reconocer y extraer texto manuscrito o impreso de una imagen de forma asincrónica y ver la imagen y los resultados. La página del escenario usa los métodos siguientes, en función del origen de la imagen:

* UploadAndRecognizeImageAsync  
  Este método se usa para las imágenes locales, en las que la imagen se debe codificar como `Stream` y enviar a Computer Vision mediante una llamada al método `RecognizeAsync` y la transferencia de un delegado parametrizado para el método `ComputerVisionClient.RecognizeTextInStreamAsync`.
* RecognizeUrlAsync  
  Este método se usa para las imágenes remotas, en las que la URL de la imagen se envía a Computer Vision mediante una llamada al método `RecognizeAsync` y la transferencia de un delegado parametrizado para el método `ComputerVisionClient.RecognizeTextAsync`.
* RecognizeAsync Este método administra la llamada asincrónica para los métodos `UploadAndRecognizeImageAsync` y `RecognizeUrlAsync`, así como el sondeo de resultados mediante una llamada al método `ComputerVisionClient.GetTextOperationResultAsync`.

A diferencia de los otros escenarios incluidos en la aplicación de ejemplo de Computer Vision, este escenario es asincrónico, ya que se llama a un método para iniciar el proceso, pero se llama a otro método para comprobar el estado y devolver los resultados de ese proceso. El flujo lógico de este escenario es un poco distinto del de los otros escenarios.

El método `UploadAndRecognizeImageAsync` abre el archivo local especificado en `imageFilePath` para la lectura como `Stream` y, a continuación, realiza una llamada al método `RecognizeAsync`, en que transfiere:

* Una expresión lambda para un delegado asincrónico parametrizado del método `ComputerVisionClient.RecognizeTextInStreamAsync`, con el elemento `Stream` para el archivo y el modo de reconocimiento como parámetros, en `GetHeadersAsyncFunc`.
* Una expresión lambda para un delegado para obtener el valor de encabezado de respuesta `Operation-Location`, en `GetOperationUrlFunc`.

El método `RecognizeUrlAsync` realiza una llamada al método `RecognizeAsync`, en que transfiere:

* Una expresión lambda para un delegado asincrónico parametrizado del método `ComputerVisionClient.RecognizeTextAsync`, con la URL de la imagen remota y el modo de reconocimiento como parámetros, en `GetHeadersAsyncFunc`.
* Una expresión lambda para un delegado para obtener el valor de encabezado de respuesta `Operation-Location`, en `GetOperationUrlFunc`.

Cuando el método `RecognizeAsync` se completa, los métodos `UploadAndRecognizeImageAsync` y `RecognizeUrlAsync` devuelven el resultado como una instancia de `TextOperationResult`. Los métodos heredados de la clase `ImageScenarioPage` presentan los resultados devueltos en la página del escenario.

El método `RecognizeAsync` realiza una llamada al delegado parametrizado para el método `ComputerVisionClient.RecognizeTextInStreamAsync` o `ComputerVisionClient.RecognizeTextAsync` transferido en `GetHeadersAsyncFunc` y espera la respuesta. A continuación, el método llama al delegado transferido en `GetOperationUrlFunc` para obtener el valor del encabezado de respuesta `Operation-Location` de la respuesta. Este valor es la dirección URL utilizada para recuperar los resultados del método transferido en `GetHeadersAsyncFunc` de Computer Vision.

A continuación, el método `RecognizeAsync` realiza una llamada al método `ComputerVisionClient.GetTextOperationResultAsync`, en que transfiere la URL recuperada del encabezado de respuesta `Operation-Location`, a fin de obtener el estado y el resultado del método transferido en `GetHeadersAsyncFunc`. Si el estado no indica que el método se completó, de manera correcta o incorrecta, el método `RecognizeAsync` realiza una llamada a `ComputerVisionClient.GetTextOperationResultAsync` 3 veces más, con un tiempo de espera de 3 segundos entre las llamadas. El método `RecognizeAsync` devuelve los resultados al método que lo llamó.

## <a name="explore-the-get-thumbnail-scenario"></a>Explorar el escenario Get Thumbnail

La página ThumbnailPage.xaml administra este escenario. Puede indicar si quiere usar el recorte inteligente, especificar la altura y la anchura que quiera a fin de generar una miniatura de una imagen y ver la imagen y los resultados. La página del escenario usa los métodos siguientes, en función del origen de la imagen:

* UploadAndThumbnailImageAsync  
  Este método se usa para las imágenes locales, en las que la imagen se debe codificar como `Stream` y enviar a Computer Vision mediante una llamada al método `ComputerVisionClient.GenerateThumbnailInStreamAsync`.
* ThumbnailUrlAsync  
  Este método se usa para las imágenes remotas, en las que la URL de la imagen se envía a Computer Vision mediante una llamada al método `ComputerVisionClient.GenerateThumbnailAsync`.

El método `UploadAndThumbnailImageAsync` crea una nueva instancia de `ComputerVisionClient` mediante la dirección URL del punto de conexión y la clave de suscripción especificadas. Dado que la aplicación de ejemplo analiza una imagen local, debe enviar el contenido de esa imagen a Computer Vision. Abre el archivo local especificado en `imageFilePath` para la lectura como `Stream`. Realiza una llamada al método `ComputerVisionClient.GenerateThumbnailInStreamAsync`, en que se transfiere la anchura, la altura, el elemento `Stream` para el archivo y si se utiliza el recorte inteligente y, a continuación, devuelve el resultado como `Stream`. Los métodos heredados de la clase `ImageScenarioPage` presentan los resultados devueltos en la página del escenario.

El método `RecognizeUrlAsync` crea una nueva instancia de `ComputerVisionClient` mediante la dirección URL del punto de conexión y la clave de suscripción especificadas. Realiza una llamada al método `ComputerVisionClient.GenerateThumbnailAsync`, en que se transfiere la anchura, la altura, la URL de la imagen y si se utiliza el recorte inteligente y, a continuación, devuelve el resultado como `Stream`. Los métodos heredados de la clase `ImageScenarioPage` presentan los resultados devueltos en la página del escenario.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite la carpeta en que ha clonado el repositorio `Microsoft/Cognitive-Vision-Windows`, elimínela. Si optó por usar las imágenes de ejemplo, elimine también la carpeta en la que clonó el repositorio `Microsoft/Cognitive-Face-Windows`.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción al servicio Face](../../Face/Tutorials/FaceAPIinCSharpTutorial.md)
