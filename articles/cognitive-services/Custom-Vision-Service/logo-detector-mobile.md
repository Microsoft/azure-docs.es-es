---
title: 'Tutorial: Uso del detector de logotipos personalizados para reconocer servicios de Azure: Custom Vision'
titleSuffix: Azure Cognitive Services
description: En este tutorial examinará una aplicación de ejemplo que usa Custom Vision como parte de un escenario de detección de logotipos. Obtenga información acerca de la forma en que Custom Vision se usa con otros componentes para proporcionar una aplicación de un extremo a otro.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: tutorial
ms.date: 08/05/2020
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 98d94e9544e75f762d4532101a92d14106d6a575
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88931801"
---
# <a name="tutorial-recognize-azure-service-logos-in-camera-pictures"></a>Tutorial: Reconocimiento de logotipos de servicios de Azure imágenes de cámara

En este tutorial examinará una aplicación de ejemplo que usa Custom Vision como parte de un escenario mayor. La aplicación AI Visual Provision es una aplicación de Xamarin.Forms para plataformas móviles que analiza imágenes de cámara de logotipos de servicios de Azure y, después, implementa dichos servicios en la cuenta de Azure del usuario. Aquí obtendrá información acerca de cómo usar Custom Vision en coordinación con otros componentes para proporcionar una útil aplicación de un extremo a otro. Puede ejecutar usted mismo toda la aplicación o bien completar solamente la parte de Custom Vision de la instalación y explorar la forma en que la aplicación lo usa.

Este tutorial le mostrará cómo:

> [!div class="checklist"]
> - Crear un detector de objetos personalizado que reconozca los logotipos de los servicios de Azure.
> - Conectar una aplicación a Azure Computer Vision y Custom Vision.
> - Crear una cuenta de entidad de servicio de Azure para implementar servicios de Azure desde la aplicación.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar. 

## <a name="prerequisites"></a>Requisitos previos

- [Visual Studio 2017 o cualquier versión posterior](https://www.visualstudio.com/downloads/)
- Carga de trabajo Xamarin para Visual Studio (consulte [Instalación de Xamarin](https://docs.microsoft.com/xamarin/cross-platform/get-started/installation/windows))
- Un emulador de iOS o de Android para Visual Studio
- La [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli-windows?view=azure-cli-latest) (opcional)

## <a name="get-the-source-code"></a>Obtención del código fuente

Si desea usar la aplicación web que se proporciona, clone o descargue el código fuente de la aplicación del repositorio [AI Visual Provision](https://github.com/Microsoft/AIVisualProvision) de GitHub. Abra el archivo *Source/VisualProvision.sln* en Visual Studio. Más adelante podrá editar algunos de los archivos de proyecto para que pueda ejecutar la aplicación.

## <a name="create-an-object-detector"></a>Creación de un detector de objetos

Inicie sesión en el [sitio web de Custom Vision](https://customvision.ai/) y cree un proyecto. Especifique un proyecto de detección de objetos y use el dominio Logo, ya que esto permitirá al servicio usar un algoritmo optimizado para la detección de logotipos. 

![Ventana del nuevo proyecto en el sitio web de Custom Vision en el explorador Chrome](media/azure-logo-tutorial/new-project.png)

## <a name="upload-and-tag-images"></a>Carga y etiquetado de imágenes

A continuación, entrene el algoritmo de detección de logotipos, para lo que debe cargar imágenes de logotipos de servicios de Azure y etiquetarlas manualmente. El repositorio AIVisualProvision incluye un conjunto de imágenes de entrenamiento que puede usar. En el sitio web, seleccione el botón **Agregar imágenes** situado en la pestaña **Training Images** (Imágenes para entrenamiento). A continuación, vaya a la carpeta **Documentos/Imágenes/Training_DataSet** del repositorio. Tendrá que etiquetar manualmente los logotipos en cada imagen, por lo que, si solo va a probar este proyecto, es posible que desee cargar solo un subconjunto de las imágenes. Cargue al menos 15 instancias de cada etiqueta que vaya a usar.

Una vez que haya cargado las imágenes de entrenamiento, seleccione la primera en la pantalla. Aparece una ventana de etiquetado. Dibuje cuadros y asigne al logotipo de cada imagen. 

![Logotipo de etiquetado en el sitio Web de Custom Vision](media/azure-logo-tutorial/tag-logos.png)

La aplicación está configurada para trabajar con cadenas de etiqueta concretas. Encontrará las definiciones en el archivo *Source\VisualProvision\Services\Recognition\RecognitionService.cs*:

[!code-csharp[Tag definitions](~/AIVisualProvision/Source/VisualProvision/Services/Recognition/RecognitionService.cs?name=snippet_constants)]

Después de etiquetar una imagen, vaya a la derecha para etiquetar siguiente. Cierre la ventana etiquetada cuando haya terminado.

## <a name="train-the-object-detector"></a>Entrenamiento del detector de objetos

En el panel izquierdo, en el parámetro **Tags** (Etiquetas), seleccione **Tagged** (Con etiqueta) para mostrar las imágenes. Luego, seleccione el botón verde de la parte superior de la página para entrenar el modelo. De esta forma se enseña al algoritmo a reconocer las mismas en las imágenes nuevas. También probará el modelo en algunas de las imágenes existentes para generar puntuaciones de precisión.

![El sitio Web de Custom Vision, en la pestaña Training Images (Imágenes para entrenamiento). En esta captura de pantalla, se describe el botón Train (Entrenar)](media/azure-logo-tutorial/train-model.png)

## <a name="get-the-prediction-url"></a>Obtención de la dirección URL de predicción

Una vez que el modelo esté entrenado, estará preparado para integrarlo en la aplicación. Deberá obtener la dirección URL del punto de conexión (la dirección del modelo que la aplicación consultará) y la clave de predicción (para conceder a la aplicación acceso a las solicitudes de predicción). En la pestaña **Performance** (Rendimiento), seleccione el botón **Prediction URL** (Dirección URL de predicción) de la parte superior de la página.

![El sitio Web de Custom Vision con una ventana de Prediction API, que muestra una dirección URL y la clave de API](media/azure-logo-tutorial/cusvis-endpoint.png)

Copie la dirección URL del punto de conexión y el valor **Prediction-Key** en los campos adecuados del archivo  *Source\VisualProvision\AppSettings.cs*:

[!code-csharp[Custom Vision fields](~/AIVisualProvision/Source/VisualProvision/AppSettings.cs?name=snippet_cusvis_keys)]

## <a name="examine-custom-vision-usage"></a>Examen del uso de Custom Vision

Abra el archivo *Source/VisualProvision/Services/Recognition/CustomVisionService.cs* para ver el modo en que la aplicación usa la clave de Custom Vision y la dirección URL del punto de conexión. El método **PredictImageContentsAsync** toma una secuencia de bytes de un archivo de imagen junto con un token de cancelación (para la administración de tareas asincrónicas), llama a Custom Vision Prediction API y devuelve el resultado de la predicción. 

[!code-csharp[Custom Vision fields](~/AIVisualProvision/Source/VisualProvision/Services/Recognition/CustomVisionService.cs?name=snippet_prediction)]

Este resultado adopta la forma de una instancia de **PredictionResult** que, a su vez, contiene una lista de instancias de **Prediction** (Predicción). Una **predicción** contiene una etiqueta detectada y la ubicación de su rectángulo delimitador en la imagen.

[!code-csharp[Custom Vision fields](~/AIVisualProvision/Source/VisualProvision/Services/Recognition/Prediction.cs?name=snippet_prediction_class)]

Para más información acerca de cómo la aplicación trata estos datos, empiece con el método **GetResourcesAsync**. Este método se define en el archivo *Source/VisualProvision/Services/Recognition/RecognitionService.cs*.  

## <a name="add-computer-vision"></a>Incorporación de Computer Vision

La parte de Custom Vision del tutorial está completa. Si desea ejecutar la aplicación, deberá integrar el servicio Computer Vision también. La aplicación usa la característica de reconocimiento de texto de Computer Vision para complementar el proceso de detección del logotipo. Un logotipo de Azure puede ser reconocido por su apariencia *o* por el texto impreso cerca de ella. A diferencia de los modelos de Custom Vision, Computer Vision está entrenado previamente para realizar determinadas operaciones en las imágenes o los vídeos.

Suscríbase al servicio Computer Vision para obtener una clave y una dirección URL del punto de conexión. Para obtener ayuda con este paso, consulte [Obtención de las claves de suscripción](https://docs.microsoft.com/azure/cognitive-services/computer-vision/vision-api-how-to-topics/howtosubscribe).

![El servicio Computer Vision en Azure Portal, con el menú de inicio rápido seleccionado. Se describe un vínculo para las claves, como es la dirección URL del punto de conexión de API](media/azure-logo-tutorial/comvis-keys.png)

A continuación, abra el archivo *Source\VisualProvision\AppSettings.cs* y rellene las variables `ComputerVisionEndpoint` y `ComputerVisionKey` con los valores correctos.

[!code-csharp[Computer Vision fields](~/AIVisualProvision/Source/VisualProvision/AppSettings.cs?name=snippet_comvis_keys)]

## <a name="create-a-service-principal"></a>Creación de una entidad de servicio

La aplicación requiere una cuenta de entidad de servicio de Azure para implementar los servicios en su suscripción de Azure. Las entidades de servicio le permiten delegar permisos concretos en una aplicación mediante el control de acceso basado en roles. Si desea más información, consulte la [guía de las entidades de servicio](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-create-service-principals).

Las entidades de servicio se pueden crear mediante Azure Cloud Shell o mediante la CLI de Azure, como se indica aquí. Para comenzar, inicie sesión y seleccione la suscripción que desea usar.

```azurecli
az login
az account list
az account set --subscription "<subscription name or subscription id>"
```

A continuación, cree la entidad de servicio. (Este proceso podría tardar algún tiempo en finalizar).

```azurecli
az ad sp create-for-rbac --name <servicePrincipalName> --password <yourSPStrongPassword>
```

Cuando se haya completado correctamente, debería ver el siguiente código JSON de salida, incluidas las credenciales necesarias.

```json
{
  "clientId": "(...)",
  "clientSecret": "(...)",
  "subscriptionId": "(...)",
  "tenantId": "(...)",
  ...
}
```

Tome nota de los valores de `clientId` y `tenantId`. Agréguelos a los campos pertinentes del archivo *Source\VisualProvision\AppSettings.cs*.

[!code-csharp[Computer Vision fields](~/AIVisualProvision/Source/VisualProvision/AppSettings.cs?name=snippet_serviceprincipal)]

## <a name="run-the-app"></a>Ejecución la aplicación

Ya ha dado a la aplicación acceso a:

- Un modelo de Custom Vision entrenado
- El servicio Computer Vision
- Una cuenta de entidad de servicio

Para ejecutar la aplicación, siga estos pasos:

1. En el Explorador de soluciones de Visual Studio, seleccione el proyecto **VisualProvision.Android** o el proyecto **VisualProvision.iOS**. Elija el emulador o dispositivo móvil conectado un correspondientes en el menú desplegable, en la barra de herramientas principal. Después, ejecute la aplicación.

    > [!NOTE]
    > Necesitará un dispositivo MacOS para ejecutar un emulador de iOS.

1. En la primera pantalla, escriba su identificador de cliente, identificador de inquilino y contraseña de la entidad de servicio. Seleccione el botón **Login** (Inicio de sesión).

    > [!NOTE]
    > En algunos emuladores, es posible que el botón **Login** (Inicio de sesión) no se active en este paso. En ese caso, detenga la aplicación, abra el archivo *Source/VisualProvision/Pages/LoginPage.xaml*, busque el elemento `Button` con la etiqueta **LOGIN BUTTON**, quite la línea y, después, ejecute de nuevo la aplicación.
    >  ```xaml
    >  IsEnabled="{Binding IsValid}"
    >  ```
    
    ![La pantalla de la aplicación que muestra los campos de las credenciales de la entidad de servicio](media/azure-logo-tutorial/app-credentials.png)

1. En la pantalla siguiente, seleccione su suscripción de Azure en el cuadro desplegable. (Este menú debe contener todas las suscripciones a la que la entidad de servicio tiene acceso). Seleccione el botón **Continue** (Continuar). En este momento, la aplicación puede pedirle que conceda acceso a los dispositivos a la cámara y al almacenamiento de fotografías. Conceda los permisos de acceso.

    ![La pantalla de la aplicación que muestra un campo desplegable para la suscripción de Azure de destino](media/azure-logo-tutorial/app-az-subscription.png)


1. Se activará la cámara del dispositivo. Haga una foto del logotipo de uno de los servicios de Azure que ha entrenado. Debería aparecer una ventana de implementación en la que se le solicite que seleccione la región y el grupo de recursos de los nuevos servicios (tal como lo haría si fuera a implementarlos desde Azure Portal). 

    ![Una pantalla de la cámara de un smartphone en la que se ven dos recortes de papel de logotipos de Azure](media/azure-logo-tutorial/app-camera-capture.png)

    ![Una pantalla de la aplicación con los campos de la región de implementación y la entrada del grupo de recursos](media/azure-logo-tutorial/app-deployment-options.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ha seguido todos los pasos de este escenario y ha usado la aplicación para implementar los servicios de Azure en su cuenta, vaya a [Azure Portal](https://ms.portal.azure.com/). Allí, cancele los servicios que no desee utilizar.

Además, si planea crear un proyecto de detección de objetos propio con Custom Vision, es posible que desee eliminar el proyecto de detección de logotipos que ha creado en este tutorial. Una suscripción gratuita de Custom Vision sirve solo para dos proyectos. Para eliminar el proyecto de detección de logotipo, en el [sitio Web de Custom Vision](https://customvision.ai), abra **Proyectos** y, a continuación, seleccione el icono de Papelera en **Mi nuevo proyecto**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha configurado y explorado una aplicación de Xamarin.Forms con todas las características que utiliza el servicio Custom Vision para detectar logotipos en las imágenes de la cámara de un móvil. A continuación, puede aprender los procedimientos recomendados para crear un modelo de Custom Vision, con el fin de que cuando cree para su propia aplicación sea más eficaz y preciso.

> [!div class="nextstepaction"]
> [Mejora del clasificador](getting-started-improving-your-classifier.md)
