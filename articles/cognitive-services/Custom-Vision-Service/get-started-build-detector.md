---
title: 'Inicio rápido: Creación de un detector de objetos en Custom Vision Service'
titleSuffix: Azure Cognitive Services
description: En este inicio rápido, aprenderá a usar el sitio web de Custom Vision para crear un modelo de clasificación de imágenes.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: quickstart
ms.date: 08/05/2020
ms.author: pafarley
ms.openlocfilehash: 01a5714cb78959a982ad73deb401edbafeebe8f9
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87834463"
---
# <a name="quickstart-how-to-build-an-object-detector-with-custom-vision"></a>Inicio rápido: Creación de un detector de objetos con Custom Vision

En este inicio rápido, aprenderá a crear un detector de objetos mediante el sitio web de Custom Vision. Una vez creado un modelo de detector, puede usar el servicio de Custom Vision para la detección de objetos.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

- Un conjunto de imágenes con el que entrenar el modelo de detector. Puede usar el conjunto de [imágenes de ejemplo](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples/tree/master/samples/vision/images) en GitHub. O bien, puede elegir sus propias imágenes con las sugerencias que se indican a continuación.

## <a name="create-custom-vision-resources-in-the-azure-portal"></a>Creación de recursos de Custom Vision en Azure Portal

[!INCLUDE [create-resources](includes/create-resources.md)]

## <a name="create-a-new-project"></a>Creación de un nuevo proyecto

En el explorador web, vaya a la [página web de Custom Vision](https://customvision.ai) y seleccione __Sign in__ (Iniciar sesión). Inicie sesión con la misma cuenta que usó para iniciar sesión en Azure Portal.

![Imagen de la página de inicio de sesión](./media/browser-home.png)


1. Para crear su primer proyecto, seleccione **New Project** (Nuevo proyecto). Aparece el cuadro de diálogo **Create new project** (Crear nuevo proyecto).

    ![Este cuadro de diálogo contiene campos de nombre, descripción y dominios.](./media/get-started-build-detector/new-project.png)

1. Escriba un nombre y una descripción para el proyecto. Después, seleccione un grupo de recursos. Si la cuenta con la que ha iniciado sesión está asociada a una cuenta de Azure, aparecerá el menú desplegable Resource Group (Grupo de recursos) que incluye un recurso de Custom Vision Service. 

   > [!NOTE]
   > Si no hay ningún grupo de recursos disponible, confirme que ha iniciado sesión en [customvision.ai](https://customvision.ai) con la misma cuenta que usó para iniciar sesión en [Azure Portal](https://portal.azure.com/). Además, confirme que el directorio seleccionado en el portal de Custom Vision es el mismo que el de Azure Portal donde se encuentran los recursos de Custom Vision. En ambos sitios, puede seleccionar el directorio en el menú de cuentas desplegable de la esquina superior derecha de la pantalla. 

1. En __Project Types__ (Tipos de proyecto), seleccione __Object Detection__ (Detección de objetos).

1. A continuación, seleccione uno de los dominios disponibles. Cada dominio optimiza el detector para determinados tipos de imágenes, como se describe en la tabla siguiente. Puede cambiar el dominio más adelante si lo desea.

    |Domain|Propósito|
    |---|---|
    |__General__| Optimizado para una amplia variedad de tareas de detección de objetos. Si ninguno de los otros dominios es adecuado, o no está seguro de qué dominio elegir, seleccione el dominio genérico. |
    |__Logotipo__|Optimizado para buscar logotipos de marca en imágenes.|
    |__Productos en las estanterías__|Optimizado para detectar y clasificar los productos que están en las estanterías.|
    |__Dominios compactos__| Optimizados para las restricciones de detección de objetos en tiempo real en dispositivos móviles. Los modelos generados por los dominios compactos se pueden exportar para ejecutarse localmente.|

1. Por último, seleccione __Create project__ (Crear proyecto).

## <a name="choose-training-images"></a>Elección de las imágenes de entrenamiento

[!INCLUDE [choose training images](includes/choose-training-images.md)]

## <a name="upload-and-tag-images"></a>Carga y etiquetado de imágenes

En esta sección cargará y etiquetará manualmente las imágenes para ayudar a entrenar el detector. 

1. Para agregar imágenes, haga clic en el botón __Add images__ (Agregar imágenes) y seleccione __Browse local files__ (Examinar archivos locales). Seleccione __Open__ (Abrir) para cargar las imágenes.

    ![El control para agregar imágenes se muestra en la parte superior izquierda y como un botón en la parte inferior central.](./media/get-started-build-detector/add-images.png)

1. Verá las imágenes cargadas en la sección **Untagged** (Sin etiqueta) de la interfaz de usuario. El siguiente paso es etiquetar manualmente los objetos que quiere que aprenda a reconocer el detector. Haga clic en la primera imagen para abrir la ventana del cuadro de diálogo de etiquetado. 

    ![Imágenes cargadas en la sección Untagged (Sin etiqueta)](./media/get-started-build-detector/images-untagged.png)

1. Haga clic y arrastre un rectángulo alrededor del objeto de la imagen. A continuación, escriba un nuevo nombre de etiqueta con el botón **+** o seleccione una etiqueta existente en la lista desplegable. Es muy importante etiquetar cada instancia de los objetos que quiera detectar, porque el detector usa el área de fondo sin etiqueta como ejemplo negativo en el entrenamiento. Cuando haya terminado de etiquetar, haga clic en la flecha de la derecha para guardar las etiquetas y pasar a la siguiente imagen.

    ![Etiquetado de un objeto con una selección rectangular](./media/get-started-build-detector/image-tagging.png)

Para cargar otro conjunto de imágenes, vuelva a la parte superior de esta sección y repita los pasos.

## <a name="train-the-detector"></a>Entrenamiento del detector

Para entrenar el modelo de detector, seleccione el botón **Train** (Entrenar). El detector usa todas las imágenes actuales y sus etiquetas para crear un modelo que identifique cada objeto etiquetado.

![Botón Train (Entrenar) en la parte superior derecha de la barra de herramientas del encabezado de la página web](./media/getting-started-build-a-classifier/train01.png)

El proceso de entrenamiento solo debe llevar unos minutos. Durante este tiempo, se muestra información sobre el proceso de entrenamiento en la pestaña **Performance** (Rendimiento).

![Ventana del explorador con un cuadro de diálogo de entrenamiento en la sección principal](./media/get-started-build-detector/training.png)

## <a name="evaluate-the-detector"></a>Evaluación del detector

Una vez finalizado el entrenamiento, se calcula y se muestra el rendimiento del modelo. Custom Vision Service usa las imágenes que ha enviado para entrenamiento para calcular la precisión, la coincidencia y la precisión media. La precisión y la coincidencia son dos medidas diferentes de la eficacia de un detector:

- La **precisión** indica la fracción de las clasificaciones identificadas que fueron correctas. Por ejemplo, si el modelo identificó 100 imágenes como perros y 99 de ellas eran realmente de perros, la precisión sería del 99 %.
- La **coincidencia** indica la fracción de las clasificaciones reales que se identificaron correctamente. Por ejemplo, si había realmente 100 imágenes de manzanas y el modelo identificó 80 como manzanas, la coincidencia sería del 80 %.

![Los resultados de entrenamiento muestran la precisión y la coincidencia generales, así como la precisión media.](./media/get-started-build-detector/trained-performance.png)

### <a name="probability-threshold"></a>Probability Threshold (Umbral de probabilidad)

[!INCLUDE [probability threshold](includes/probability-threshold.md)]

## <a name="manage-training-iterations"></a>Administración de iteraciones de entrenamiento

Cada vez que entrena al detector, se crea una _iteración_ con sus propias métricas de rendimiento actualizadas. Puede ver todas las iteraciones en el panel izquierdo de la pestaña **Performance** (Rendimiento). En el panel izquierdo encontrará también el botón **Delete** (Eliminar), que puede usar para eliminar una iteración si está obsoleta. Cuando se elimina una iteración, elimina las imágenes que están asociadas exclusivamente a ella.

Consulte [Uso del modelo con Prediction API](./use-prediction-api.md) para aprender a acceder a los modelos entrenados mediante programación.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear y entrenar un modelo de detector de objetos mediante el sitio web de Custom Vision. A continuación, obtenga más información sobre el proceso iterativo de mejora del modelo.

> [!div class="nextstepaction"]
> [Prueba y reentrenamiento del modelo](test-your-model.md)

