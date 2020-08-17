---
title: 'Inicio rápido: Biblioteca cliente de Computer Vision para .NET'
description: En este inicio rápido, se muestra una introducción a la biblioteca cliente de Computer Vision para .NET.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 12/05/2019
ms.author: pafarley
ms.openlocfilehash: c1406b5e7297b1d48b23d9dfa684e0d76b68139f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87103014"
---
<a name="HOLTop"></a>

[Documentación de referencia](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/computervision?view=azure-dotnet) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Vision.ComputerVision) | [Paquete (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.ComputerVision/) | [Ejemplos](https://azure.microsoft.com/resources/samples/?service=cognitive-services&term=vision&sort=0)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* La versión más reciente del [SDK de .NET Core](https://dotnet.microsoft.com/download/).
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="Creación de un recurso de Computer Vision"  target="_blank">cree un recurso de Computer Vision <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación al servicio Computer Vision. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* [Cree las variables de entorno](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication) `COMPUTER_VISION_SUBSCRIPTION_KEY` y `COMPUTER_VISION_ENDPOINT` para la clave y la dirección URL del punto de conexión, respectivamente.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

Cree una nueva aplicación de consola de .NET Core en el IDE o editor que prefiera. 

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `computer-vision-quickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: *ComputerVisionQuickstart.cs*.

```dotnetcli
dotnet new console -n computer-vision-quickstart
```

Cambie el directorio a la carpeta de aplicaciones recién creada. Para compilar la aplicación:

```dotnetcli
dotnet build
```

La salida de la compilación no debe contener advertencias ni errores. 

```console
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

En el directorio del proyecto, abra el archivo *ComputerVisionQuickstart.cs* en el editor o IDE que prefiera. Agregue las siguientes directivas `using`:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_using)]

En la clase **Program** de la aplicación, cree variables para el punto de conexión y la clave de Azure del recurso.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_vars)]

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Dentro del directorio de aplicaciones, instale la biblioteca cliente de Computer Vision para .NET con el siguiente comando:

```dotnetcli
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0-preview.1
```

Si usa el IDE de Visual Studio, la biblioteca cliente estará disponible como un paquete de NuGet descargable.

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales del SDK de .NET para Computer Vision.

|Nombre|Descripción|
|---|---|
| [ComputerVisionClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.computervisionclient?view=azure-dotnet) | Esta clase es necesaria para todas las funcionalidades de Computer Vision. Cree una instancia de ella con la información de suscripción y úsela para realizar la mayoría de las operaciones con imágenes.|
|[ComputerVisionClientExtensions](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.computervisionclientextensions?view=azure-dotnet)| Esta clase contiene métodos adicionales para **ComputerVisionClient**.|
|[VisualFeatureTypes](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.models.visualfeaturetypes?view=azure-dotnet)| Esta enumeración define los diferentes tipos de análisis de imágenes que se pueden realizar en una operación de análisis estándar. Debe especificar un conjunto de valores de VisualFeatureTypes en función de sus necesidades. |

## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código se muestra cómo realizar las siguientes tareas con la biblioteca cliente de Computer Vision para .NET:

* [Autenticar el cliente](#authenticate-the-client)
* [Analizar una imagen](#analyze-an-image)
* [Leer texto manuscrito e impreso](#read-printed-and-handwritten-text)

## <a name="authenticate-the-client"></a>Autenticar el cliente

> [!NOTE]
> En este inicio rápido se da por supuesto que ha [creado variables de entorno](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication) para la clave de Computer Vision y el punto de conexión, denominadas `COMPUTER_VISION_SUBSCRIPTION_KEY` y `COMPUTER_VISION_ENDPOINT` respectivamente.

En un nuevo método, cree una instancia de un cliente con la clave y el punto de conexión. Cree un objeto **[ApiKeyServiceClientCredentials](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.apikeyserviceclientcredentials?view=azure-dotnet)** con la clave y úselo con el punto de conexión para crear un objeto **[ComputerVisionClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.computervisionclient?view=azure-dotnet)** .

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_auth)]

Es probable que desee llamar a este método en el método `Main`.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_client)]

## <a name="analyze-an-image"></a>Análisis de una imagen

El código siguiente define un método, `AnalyzeImageUrl`, que utiliza el objeto de cliente para analizar una imagen remota e imprimir los resultados. El método devuelve una descripción de texto, categorización, lista de etiquetas, caras detectadas, marcas de contenido para adultos, colores principales y tipo de imagen.

Agregue la llamada al método en el método `Main`.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_analyzeinmain)]

### <a name="set-up-test-image"></a>Configuración de una imagen de prueba

En la clase **Program**, guarde una referencia a la dirección URL de la imagen que desea analizar.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_analyze_url)]

> [!NOTE]
> También puede analizar una imagen local. Consulte el código de ejemplo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ComputerVision/ComputerVisionQuickstart.cs) para ver los escenarios que implican imágenes locales.

### <a name="specify-visual-features"></a>Especificación de características visuales

Defina el nuevo método para el análisis de imágenes. Agregue el código siguiente, que especifica qué características visuales desea extraer en el análisis. Vea la enumeración **[VisualFeatureTypes](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.models.visualfeaturetypes?view=azure-dotnet)** para obtener una lista completa.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_visualfeatures)]

Inserte cualquiera de los siguientes bloques de código en el método **AnalyzeImageUrl** para implementar sus características. No olvide agregar un corchete de cierre al final.

```csharp
}
```

### <a name="analyze"></a>Analizar

El método **AnalyzeImageAsync** devuelve un objeto **ImageAnalysis** que contiene toda la información extraída.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_analyze_call)]

En las secciones siguientes se muestra cómo analizar esta información en detalle.

### <a name="get-image-description"></a>Obtención de la descripción de la imagen

El código siguiente obtiene la lista de títulos generados para la imagen. Consulte [Descripción de imágenes](../../concept-describing-images.md) para más detalles.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_describe)]

### <a name="get-image-category"></a>Obtención de la categoría de imagen

El código siguiente obtiene la categoría detectada de la imagen. Consulte [Categorización de imágenes](../../concept-categorizing-images.md) para más detalles.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_categorize)]

### <a name="get-image-tags"></a>Obtención de etiquetas de imagen

El código siguiente obtiene el conjunto de las etiquetas detectadas en la imagen. Consulte [Etiquetas de contenido](../../concept-tagging-images.md) para más detalles.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_tags)]

### <a name="detect-objects"></a>Detección de objetos

El código siguiente detecta objetos comunes en la imagen y los imprime en la consola. Consulte [Detección de objetos](../../concept-object-detection.md) para más información.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_objects)]

### <a name="detect-brands"></a>Detección de marcas

El código siguiente detecta marcas corporativas y logotipos en la imagen y los imprime en la consola. Consulte [Detección de marcas](../../concept-brand-detection.md) para más información.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_brands)]

### <a name="detect-faces"></a>Detección de caras

El código siguiente devuelve las caras detectadas en la imagen con sus coordenadas de rectángulo y selecciona los atributos de cara. Consulte [Detección de caras](../../concept-detecting-faces.md) para más información.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_faces)]

### <a name="detect-adult-racy-or-gory-content"></a>Detección de contenido para adultos, explícito o sangriento

El siguiente código imprime la presencia detectada de contenido para adultos en la imagen. Para más información, consulte [Contenido para adultos, subido de tono y sangriento](../../concept-detecting-adult-content.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_adult)]

### <a name="get-image-color-scheme"></a>Obtención de la combinación de colores de imagen

El código siguiente imprime los atributos de color detectados en la imagen, como los colores dominantes y el color de énfasis. Consulte [Combinaciones de colores](../../concept-detecting-color-schemes.md) para más detalles.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_color)]

### <a name="get-domain-specific-content"></a>Obtención de contenido específico del dominio

Computer Vision puede usar modelos especializados para realizar análisis adicionales en las imágenes. Consulte [Contenido específico del dominio](../../concept-detecting-domain-content.md) para más detalles. 

En el código siguiente se analizan los datos sobre las celebridades detectadas en la imagen.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_celebs)]

En el código siguiente se analizan los datos sobre los paisajes detectados en la imagen.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_landmarks)]

### <a name="get-the-image-type"></a>Obtención del tipo de imagen

El código siguiente imprime información sobre el tipo de imagen (si es una imagen prediseñada o dibujo lineal).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_type)]

## <a name="read-printed-and-handwritten-text"></a>Lectura de texto manuscrito e impreso

Computer Vision puede leer texto visible de una imagen y convertirlo en un flujo de caracteres. Para obtener más información sobre el reconocimiento de texto, consulte la documentación conceptual sobre el [reconocimiento óptico de caracteres (OCR)](../../concept-recognizing-text.md#read-api). El código de esta sección define un método, `BatchReadFileUrl`, que usa el objeto de cliente para detectar y extraer el texto de la imagen.

Agregue la llamada al método en el método `Main`.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_extracttextinmain)]

### <a name="set-up-test-image"></a>Configuración de una imagen de prueba

En la clase **Program**, guarde una referencia de la dirección URL de la imagen de la que quiere extraer texto. Este fragmento de código incluye imágenes de ejemplo para texto impreso y manuscrito.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_extracttext_url)]

> [!NOTE]
> También puede extraer texto de una imagen local. Consulte el código de ejemplo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ComputerVision/ComputerVisionQuickstart.cs) para ver los escenarios que implican imágenes locales.

### <a name="call-the-read-api"></a>Llamada a la API Read

Defina el nuevo método para leer texto. Agregue el código siguiente, que llama al método **ReadAsync** para la imagen especificada. Esto devuelve un identificador de operación e inicia un proceso asincrónico para leer el contenido de la imagen.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_extract_call)]

### <a name="get-read-results"></a>Obtención de resultados de lectura

A continuación, obtenga el identificador de operación devuelto por la llamada a **ReadAsync** y úselo para consultar los resultados de la operación en el servicio. El código siguiente comprueba la operación a intervalos de un segundo hasta que se devuelven los resultados. A continuación, imprime los datos de texto extraídos en la consola.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_extract_response)]

### <a name="display-read-results"></a>Visualización de resultados de lectura

Agregue el código siguiente para analizar y mostrar los datos de texto recuperados y finalice la definición del método.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_extract_display)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```dotnetcli
dotnet run
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
>[Referencia de la API Computer Vision (.NET)](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/computervision?view=azure-dotnet)

* [¿Qué es Computer Vision?](../../Home.md)
* El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ComputerVision/ComputerVisionQuickstart.cs).
