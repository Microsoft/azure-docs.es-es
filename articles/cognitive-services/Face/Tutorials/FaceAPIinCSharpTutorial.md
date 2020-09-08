---
title: 'Tutorial: Detección y presentación de los datos de las caras en una imagen mediante el SDK de .NET'
titleSuffix: Azure Cognitive Services
description: En este tutorial, creará una aplicación Windows que usa el servicio Face para detectar y enmarcar las caras en una imagen.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: tutorial
ms.date: 08/17/2020
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 13bbb5e006f725ff0b75a5b86aee414f84a80dcf
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936306"
---
# <a name="tutorial-create-a-windows-presentation-framework-wpf-app-to-display-face-data-in-an-image"></a>Tutorial: Creación de una aplicación de Windows Presentation Framework (WPF) para mostrar datos de las caras de una imagen

En este tutorial, aprenderá a usar el servicio Azure Face mediante el SDK de cliente para .NET, para detectar caras en una imagen y, después, presentar dichos datos en la interfaz de usuario. Creará una aplicación de WPF que detecta caras, dibuja un marco alrededor de cada una de ellas y muestra una descripción de la cara en la barra de estado. 

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> - Crear una aplicación WPF
> - Instalación de la biblioteca cliente de Face
> - Usar la biblioteca cliente para detectar caras en una imagen
> - Dibujar un marco alrededor de cada cara detectada
> - Mostrar una descripción de la cara resaltada en la barra de estado

![Captura de pantalla que muestra las caras detectadas enmarcadas en rectángulos](../Images/getting-started-cs-detected.png)

El código de ejemplo completo está disponible en el repositorio del [ejemplo de Cognitive Face CSharp](https://github.com/Azure-Samples/Cognitive-Face-CSharp-sample) de GitHub.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar. 


## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="Creación de un recurso de Face"  target="_blank">cree un recurso de Face <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Face API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* [Cree variables de entorno](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication) para la cadena de punto de conexión del servicio y la clave denominadas `FACE_SUBSCRIPTION_KEY` y `FACE_ENDPOINT`, respectivamente.
- Cualquier edición de [Visual Studio](https://www.visualstudio.com/downloads/).

## <a name="create-the-visual-studio-project"></a>Creación del proyecto de Visual Studio

Siga estos pasos para crear un nuevo proyecto de aplicación WPF.

1. En Visual Studio, abra el cuadro de diálogo Nuevo proyecto. Expanda **Instalado**, luego, **Visual C#**  y, después, seleccione **Aplicación WPF (.NET Framework)** .
1. Asigne a la aplicación el nombre **FaceTutorial** y haga clic en **Aceptar**.
1. Obtenga los paquetes NuGet requeridos. Haga clic con el botón derecho en el proyecto en el Explorador de soluciones y seleccione **Administrar paquetes NuGet**; luego, busque e instale el siguiente paquete:
    - [Microsoft.Azure.CognitiveServices.Vision.Face 2.6.0-preview.1](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.Face/2.6.0-preview.1)

## <a name="add-the-initial-code"></a>Adición del código inicial

En esta sección, agregará el marco básico de la aplicación sin sus características específicas de la cara.

### <a name="create-the-ui"></a>Creación de la interfaz de usuario

Abra *MainWindow.xaml* y reemplace el contenido por el código siguiente; así se crea la ventana de la interfaz de usuario. Los métodos `FacePhoto_MouseMove` y `BrowseButton_Click` son controladores de eventos que definirá más adelante.

[!code-xaml[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml?name=snippet_xaml)]

### <a name="create-the-main-class"></a>Creación de la clase principal

Abra *MainWindow.xaml.cs* y agregue los espacios de nombres de la biblioteca cliente, junto con otros espacios de nombres necesarios. 

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_using)]

Luego, inserte el código siguiente en la clase **MainWindow**. Este código crea una instancia de **FaceClient** mediante la clave de suscripción y el punto de conexión.

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_mainwindow_fields)]

A continuación, agregue el constructor **MainWindow**. Comprueba la cadena de la dirección URL del punto de conexión y, después, la asocia al objeto de cliente.

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_mainwindow_constructor)]

Por último, agregue los métodos **BrowseButton_Click** y **FacePhoto_MouseMove** a la clase. Estos métodos corresponden a los controladores de eventos declarados en *MainWindow.xaml*. El método **BrowseButton_Click** crea **OpenFileDialog**, que permite al usuario seleccionar una imagen .jpg. Luego, muestra la imagen en la ventana principal. Se insertará el código restante de **BrowseButton_Click** y **FacePhoto_MouseMove** en los pasos posteriores. Tenga en cuenta también la referencia`faceList`&mdash; a una lista de objetos **DetectedFace**. En esta referencia es donde la aplicación almacenará y llamará a los datos reales de la cara.

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_browsebuttonclick_start)]

<!-- [!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_browsebuttonclick_end)] -->

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_mousemove_start)]

<!-- [!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_mousemove_end)] -->

### <a name="try-the-app"></a>Prueba de la aplicación

Presione **Iniciar** en el menú para probar la aplicación. Cuando se abra la ventana de la aplicación, haga clic en **Examinar** en la esquina inferior izquierda. Debería aparecer el cuadro de diálogo **Abrir archivo**. Seleccione una imagen del sistema de archivos y compruebe que se muestra en la ventana. Luego, cierre la aplicación y vaya al paso siguiente.

![Captura de pantalla que se muestra la imagen sin modificar de caras](../Images/getting-started-cs-ui.png)

## <a name="upload-image-and-detect-faces"></a>Carga de imagen y detección de caras

La aplicación detectará las caras mediante una llamada al método **FaceClient.Face.DetectWithStreamAsync**, que ajusta la API REST de [detección](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) para cargar una imagen local.

Inserte el siguiente método en la clase **MainWindow** debajo del método **FacePhoto_MouseMove**. Este método define una lista de los atributos de la cara que se recuperan y lee el archivo de imagen enviada en **Stream**. Luego pasa estos dos objetos a la llamada al método **DetectWithStreamAsync**.

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_uploaddetect)]

## <a name="draw-rectangles-around-faces"></a>Dibujar rectángulos alrededor de las caras

Luego, agregará el código para dibujar un rectángulo alrededor de cada cara detectada en la imagen. En la clase **MainWindow**, inserte el código siguiente al final del método **BrowseButton_Click**, después de la línea `FacePhoto.Source = bitmapSource`. Este código rellena una lista de las caras detectadas de la llamada a **UploadAndDetectFaces**. Luego traza un rectángulo alrededor de cada cara y muestra la imagen modificada en la ventana principal.

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_browsebuttonclick_mid)]

## <a name="describe-the-faces"></a>Descripción de las caras

Agregue el método siguiente a la clase **MainWindow**, debajo del método **UploadAndDetectFaces**. Este método convierte los atributos recuperados de la cara en una cadena que la describe.

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_facedesc)]

## <a name="display-the-face-description"></a>Visualización de la descripción de la cara

Agregue el siguiente código al método **FacePhoto_MouseMove**. Este controlador de eventos muestra la cadena de descripción de la cara de`faceDescriptionStatusBar` cuando el cursor pasa por encima de un rectángulo de la cara detectada.

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_mousemove_mid)]

## <a name="run-the-app"></a>Ejecución la aplicación

Ejecute la aplicación y busque una imagen que contenga una cara. Espere unos segundos para permitir que el servicio Face responda. Debe ver un rectángulo rojo en cada una de las caras de la imagen. Si mueve el ratón sobre el rectángulo de una cara, su descripción debería aparecer en la barra de estado.

![Captura de pantalla que muestra las caras detectadas enmarcadas en rectángulos](../Images/getting-started-cs-detected.png)


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido el proceso básico para usar el SDK de .NET del servicio Face y ha creado una aplicación que detecta y enmarca las caras de una imagen. Después, obtendrá más información acerca de los detalles de la detección de caras.

> [!div class="nextstepaction"]
> [Detección de caras en una imagen](../Face-API-How-to-Topics/HowtoDetectFacesinImage.md)
