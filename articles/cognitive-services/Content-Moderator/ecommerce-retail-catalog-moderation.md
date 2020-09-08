---
title: 'Tutorial: Moderación de imágenes de productos de comercio electrónico: Content Moderator'
titleSuffix: Azure Cognitive Services
description: Este tutorial le muestra cómo configurar una aplicación para analizar y clasificar las imágenes de productos con las etiquetas especificadas (mediante Azure Computer Vision y Custom Vision). Etiquete imágenes inaceptables para su revisión adicional (mediante Azure Content Moderator).
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: tutorial
ms.date: 06/29/2020
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 017dabf16384e53d924ed69f36b64050fcacb5bf
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88934793"
---
# <a name="tutorial-moderate-e-commerce-product-images-with-azure-content-moderator"></a>Tutorial: Moderación de imágenes de productos de comercio electrónico con Azure Content Moderator

En este tutorial, obtendrá información sobre cómo usar Azure Cognitive Services, que incluye Content Moderator, para clasificar y moderar imágenes de productos en un escenario de comercio electrónico. Usará Computer Vision y Custom Vision para aplicar etiquetas a las imágenes y, a continuación, creará una revisión de equipo, que combina las tecnologías basadas en aprendizaje automático de Content Moderator con equipos de revisión humana para formar un sistema de moderación inteligente.

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Regístrese en Content Moderator y cree un equipo de revisión.
> * Uso de la API de imagen de Content Moderator para detectar posible contenido para adultos o subido de tono.
> * Use el servicio Computer Vision para buscar contenido de celebridades (u otras etiquetas detectables por Computer Vision).
> * Use Custom Vision Service para detectar la presencia de banderas, juguetes y lápices (u otras etiquetas personalizadas).
> * Presente los resultados combinados para una revisión humana y la toma de decisión final.

El código de ejemplo completo está disponible en el repositorio [Samples eCommerce Catalog Moderation](https://github.com/MicrosoftContentModerator/samples-eCommerceCatalogModeration) de GitHub.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

- Una clave de suscripción de Content Moderator. Siga las instrucciones de [Creación de una cuenta de Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) para suscribirse al servicio Content Moderator y obtener su clave.
- Una clave de suscripción de Computer Vision (las mismas instrucciones que antes).
- Cualquier edición de [Visual Studio 2015 o 2017](https://www.visualstudio.com/downloads/).
- Un conjunto de imágenes para cada etiqueta que va a usar el clasificador de Custom Vision (en este caso juguetes, lápices y banderas de EE. UU.).

## <a name="create-a-review-team"></a>Creación de un equipo de revisión

Consulte la guía de inicio rápido [Cómo familiarizarse con Content Moderator](quick-start.md) para obtener instrucciones sobre cómo registrarse en la [herramienta de revisión de Content Moderator](https://contentmoderator.cognitive.microsoft.com/) y crear un equipo de revisión. Anote el valor de **Id. de equipo** en la página **Credenciales**.

## <a name="create-custom-moderation-tags"></a>Creación de etiquetas de moderación personalizadas

A continuación, cree etiquetas personalizadas en la herramienta de revisión (consulte el artículo [Etiquetas](https://docs.microsoft.com/azure/cognitive-services/content-moderator/review-tool-user-guide/tags) si necesita ayuda con este proceso). En este caso, agregaremos las siguientes etiquetas: **celebridad**, **EE. UU.** , **bandera**, **juguete** y **lápiz**. No todas las etiquetas tienen que tener categorías detectables en Computer Vision (como **celebridad**); puede agregar sus propias etiquetas personalizadas, siempre que entrene al clasificador de Custom Vision para que pueda detectarlas más adelante.

![Configuración de etiquetas personalizadas](images/tutorial-ecommerce-tags2.PNG)

## <a name="create-visual-studio-project"></a>Creación de un proyecto de Visual Studio

1. En Visual Studio, abra el cuadro de diálogo Nuevo proyecto. Expanda **Instalado**, luego, **Visual C#**  y, después, seleccione **Aplicación de consola (.NET Framework)** .
1. Asigne a la aplicación el nombre **EcommerceModeration** y haga clic en **Aceptar**.
1. Si va a agregar este proyecto a una solución existente, selecciónelo como proyecto de inicio único.

En este tutorial se resalta el código que es fundamental para el proyecto, pero no se describirán todas las líneas de código. Copie el contenido completo de _Program.cs_ del proyecto de ejemplo ([Samples eCommerce Catalog Moderation](https://github.com/MicrosoftContentModerator/samples-eCommerceCatalogModeration)) en el archivo _Program.cs_ del nuevo proyecto. Después, continúe por las siguientes secciones para obtener información sobre cómo funciona el proyecto y cómo usarlo.

## <a name="define-api-keys-and-endpoints"></a>Definición de los puntos de conexión y las claves de API

Este tutorial usa tres servicios de Cognitive Services; por consiguiente, requiere tres claves y puntos de conexión de API. Vea los campos siguientes de la clase **Program**:

[!code-csharp[define API keys and endpoint URIs](~/samples-eCommerceCatalogModeration/Fusion/Program.cs?range=21-29)]

Tendrá que actualizar los campos `___Key` con los valores de las claves de suscripción y cambiar los campos `___Uri` a las direcciones URL de punto de conexión correctas (obtendrá la clave y el punto de conexión de Computer Vision más adelante). Puede encontrar estos valores en las pestañas **Inicio rápido** de cada recurso de Azure. Rellene la parte `YOURTEAMID` del campo `ReviewUri` con el identificador del equipo de revisión que creó anteriormente. Rellenará la parte final del campo `CustomVisionUri` más adelante.

[!INCLUDE [subdomains note](../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="primary-method-calls"></a>Llamadas al método principal

Vea el siguiente código en el método **Main**, que recorre en iteración una lista de direcciones URL de imagen. Analiza cada imagen con los tres servicios diferentes, registra las etiquetas aplicadas en la matriz **ReviewTags** y, después, crea una revisión para los moderadores humanos (envía las imágenes a la herramienta de revisión de Content Moderator). En las siguientes secciones veremos estos métodos. Si lo desea, puede controlar qué imágenes se envían a revisión, utilizando para ello la matriz **ReviewTags** en una instrucción condicional para comprobar qué etiquetas se aplicaron.

[!code-csharp[Main: evaluate each image and create review](~/samples-eCommerceCatalogModeration/Fusion/Program.cs?range=53-70)]

## <a name="evaluateadultracy-method"></a>Método EvaluateAdultRacy

Consulte el método **EvaluateAdultRacy** en la clase **Program**. Este método emplea una dirección URL de imagen y una matriz de pares de clave-valor como parámetros. Llama a Image API de Content Moderator (mediante REST) para obtener las puntuaciones de las categorías "para adultos" y "subido de tono" de la imagen. Si la puntuación es superior a 0,4 (el intervalo es entre 0 y 1), establece el valor correspondiente en la matriz **ReviewTags** en **True**.

[!code-csharp[define EvaluateAdultRacy method](~/samples-eCommerceCatalogModeration/Fusion/Program.cs?range=73-113)]

## <a name="evaluatecomputervisiontags-method"></a>Método EvaluateComputerVisionTags

El siguiente método toma una dirección URL de imagen y la información de la suscripción de Computer Vision y analiza la presencia de celebridades en la imagen. Si se encuentran una o más celebridades, establece el valor correspondiente de la matriz **ReviewTags** en **True**.

[!code-csharp[define EvaluateCustomVisionTags method](~/samples-eCommerceCatalogModeration/Fusion/Program.cs?range=115-146)]

## <a name="evaluatecustomvisiontags-method"></a>Método EvaluateCustomVisionTags

A continuación, consulte el método **EvaluateCustomVisionTags**, que clasifica los productos reales; en este caso, banderas, juguetes y lápices. Siga las instrucciones de la guía [Cómo crear un clasificador](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/getting-started-build-a-classifier) para crear un clasificador de imágenes personalizado para detectar banderas, juguetes y lápices (o lo que haya elegido como etiquetas personalizadas) en las imágenes. Puede usar las imágenes de la carpeta **sample-images** carpeta del [repositorio de GitHub](https://github.com/MicrosoftContentModerator/samples-eCommerceCatalogModeration) para entrenar rápidamente algunas de las categorías de este ejemplo.

![Página web de Custom Vision con imágenes de entrenamiento de lápices, juguetes y banderas](images/tutorial-ecommerce-custom-vision.PNG)

Una vez entrenado el clasificador, obtenga la clave de predicción y la dirección URL del punto de conexión de predicción (consulte [Obtención de una clave de predicción y dirección URL](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/use-prediction-api#get-the-url-and-prediction-key) si necesita ayuda para recuperarlas) y asigne estos valores a los campos `CustomVisionKey` y `CustomVisionUri`, respectivamente. El método usa estos valores para consultar el clasificador. Si el clasificador encuentra una o más etiquetas personalizadas en la imagen, este método establece los valores correspondientes de la matriz **ReviewTags** en **True**.

[!code-csharp[define EvaluateCustomVisionTags method](~/samples-eCommerceCatalogModeration/Fusion/Program.cs?range=148-171)]

## <a name="create-reviews-for-review-tool"></a>Creación de revisiones para la herramienta de revisión

En las secciones anteriores, vio cómo la aplicación examina las imágenes entrantes en busca de contenido para adultos o subido de tono (Content Moderator), celebridades (Computer Vision) y diversos objetos (Custom Vision). A continuación, verá el método **CreateReview**, que carga las imágenes con todas las etiquetas aplicadas (pasadas como _Metadata_) en la herramienta de revisión de Content Moderator.

[!code-csharp[define CreateReview method](~/samples-eCommerceCatalogModeration/Fusion/Program.cs?range=173-196)]

Las imágenes se mostrarán en la pestaña Review (Revisión) de la [herramienta de revisión de Content Moderator](https://contentmoderator.cognitive.microsoft.com/).

![Captura de pantalla de la herramienta de revisión de Content Moderator con varias imágenes y sus etiquetas resaltadas](images/tutorial-ecommerce-content-moderator.PNG)

## <a name="submit-a-list-of-test-images"></a>Envío de una lista de imágenes de prueba

Como puede ver en el método **Main**, este programa busca un directorio "C:Test" con un archivo _Urls.txt_ que contiene una lista de direcciones URL de imagen. Cree este archivo y directorio, o cambie la ruta de acceso para que apunte al archivo de texto. A continuación, rellene este archivo con las direcciones URL de imagen que desea probar.

[!code-csharp[Main: set up test directory, read lines](~/samples-eCommerceCatalogModeration/Fusion/Program.cs?range=38-51)]

## <a name="run-the-program"></a>Ejecución del programa

Si ha seguido todos los pasos anteriores, el programa procesará cada imagen (y consultará las etiquetas correspondientes a los tres servicios) y, después, cargará las imágenes con información de etiqueta en la herramienta de revisión de Content Moderator.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, configurará un programa para analizar las imágenes de productos y etiquetarlas por tipo de producto para que un equipo de revisión pueda tomar decisiones informadas sobre la moderación de contenido. Después, obtendrá más información acerca de los detalles de la moderación de imágenes.

> [!div class="nextstepaction"]
> [Revisión de imágenes moderadas](./review-tool-user-guide/review-moderated-images.md)
