---
title: 'Uso del sitio web de Video Indexer para personalizar un modelo de persona: Azure'
titlesuffix: Azure Media Services
description: En este artículo se muestra cómo personalizar un modelo de persona con el sitio web de Video Indexer.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: anzaman
ms.openlocfilehash: 37e556ca458a5b0a171664e089d39cfb448d5f96
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65800112"
---
# <a name="customize-a-person-model-with-the-video-indexer-website"></a>Personalización de un modelo de persona con el sitio web de Video Indexer

Video Indexer es compatible con reconocimiento de celebridades para contenido de vídeo. La característica de reconocimiento de celebridades abarca aproximadamente un millón de caras basadas en orígenes de datos comúnmente solicitados, como IMDB, Wikipedia y personas con más influencia de LinkedIn. Para obtener información detallada, consulte [personalizar un modelo de persona en Video Indexer](customize-person-model-overview.md).

Puede usar el sitio web de Video Indexer para editar las caras que se detectaron en un vídeo, como se describe en este tema. También puede usar la API, como se describe en [personalizar un modelo de la persona mediante las API de](customize-person-model-with-api.md).

## <a name="central-management-of-person-models-in-your-account"></a>Administración central de modelos de persona de su cuenta

1. Para ver, editar y eliminar los modelos de persona de su cuenta, vaya al sitio Web de Video Indexer e inicie sesión.
2. Haga clic en el botón de personalización del modelo de contenido en la esquina superior derecha de la página.

    ![Personalización del modelo de contenido](./media/customize-face-model/content-model-customization.png)
3. Seleccione la pestaña de personas.

    Verá el modelo predeterminado persona en su cuenta. El modelo de persona predeterminada contiene las caras que ha editado o modificado en la información de los vídeos que no se ha especificado un modelo de persona personalizado durante la indización. 

    Si ha creado otros modelos de persona, se mostrarán en esta página también. 

    ![Personalización del modelo de contenido](./media/customize-face-model/content-model-customization-people-tab.png)

## <a name="create-a-new-person-model"></a>Creación de un modelo de persona

1. Haga clic en el **+ Agregar modelo** botón.

    ![Agregue una nueva persona](./media/customize-face-model/add-new-person.png)
2. Escriba el nombre del modelo y haga clic en el botón de verificación situada junto al nombre.

    ![Agregue una nueva persona](./media/customize-face-model/add-new-person2.png)

    Ha creado un nuevo modelo de persona. Ahora puede agregar caras al nuevo modelo de persona.
3. Haga clic en el botón de menú de lista y elija **+ agregar una persona**.

    ![Agregue una nueva persona](./media/customize-face-model/add-new-person3.png)
    
## <a name="add-a-new-person-to-a-person-model"></a>Agregar a una nueva persona a un modelo de persona

> [!NOTE]
> Video Indexer le permite agregar varias personas con el mismo nombre en un modelo de la persona. Sin embargo, se recomienda que se asigne a unque nombres a cada persona en el modelo para mayor claridad y la facilidad de uso.

1. Para agregar un nuevo aspecto para un modelo de persona, haga clic en el botón de menú de la lista junto al modelo de la persona que desea agregar la cara.
1. Haga clic en **+ agregar una persona** en el menú.

    ![Agregar una nueva cara a la persona](./media/customize-face-model/add-new-face.png)
 
    Un elemento emergente le pedirá que rellene los detalles de la persona. Escriba el nombre de la persona y haga clic en el botón de comprobación.

    ![Agregar una nueva cara a la persona](./media/customize-face-model/add-new-face2.png)
 
A continuación, puede elegir entre el Explorador de archivos o arrastrar y colocar las imágenes de la cara de la cara. Video Indexer tardará imagen estándar de todos los tipos de archivo (p. ej.: JPG, PNG etc.).

Indexador de vídeo debe ser capaz de detectar las repeticiones de esta persona en los futuros vídeos ese índice y los vídeos actuales que ya tenía indizado, mediante el modelo de persona a la que se ha agregado esta nueva cara a. Reconocimiento de la persona en los vídeos actuales puede tardar algún tiempo en aplicarse, tal como se trata de un proceso por lotes.


## <a name="rename-a-person-model"></a>Cambiar el nombre de un modelo de persona

Puede cambiar el nombre de cualquier modelo de persona en su cuenta incluida en el modelo predeterminado persona. Incluso si cambia el nombre de su modelo de la persona de forma predeterminada, todavía servirá como el modelo de la persona de forma predeterminada en su cuenta.

1. Haga clic en el botón de menú de la lista junto al modelo de la persona que desea cambiar el nombre.
2. Haga clic en **cambiar el nombre** en el menú.

    ![Cambiar el nombre de una persona](./media/customize-face-model/rename-person.png)
3. Haga clic en el nombre del modelo actual y escriba el nuevo nombre.

    ![Cambiar el nombre de una persona](./media/customize-face-model/rename-person2.png)
4. Haga clic en el botón de comprobación para el modelo que se va a cambiar.

## <a name="delete-a-person-model"></a>Eliminación de un modelo de persona

Puede eliminar cualquier modelo de la persona que creó en su cuenta. Sin embargo, no se puede eliminar el modelo de la persona de forma predeterminada.

1. Haga clic en **eliminar** en el menú.

    ![Eliminar una persona](./media/customize-face-model/delete-person.png)
    
    Se mostrará una ventana emergente y se le notificará que esta acción eliminará el modelo de persona y todas las personas y los archivos que contiene. Esta acción no se puede deshacer. 

    ![Eliminar una persona](./media/customize-face-model/delete-person2.png)
1. Si está seguro, haga clic en eliminar de nuevo.

> [!NOTE]
> Los vídeos existentes que se han indizado mediante este modelo de la persona (eliminado) no será compatible con la capacidad de actualizar los nombres de las caras que aparecen en el vídeo. Podrá editar los nombres de caras en estos vídeos solo después de volver a indizar mediante el modelo de otra persona. Si volver a indizar sin especificar un modelo de persona, se usará el modelo predeterminado. 

## <a name="manage-existing-people-in-a-person-model"></a>Administrar usuarios existentes en un modelo de persona

Para ver el contenido de cualquiera de los modelos de persona, haga clic en la flecha situada junto al nombre del modelo de persona.
La lista desplegable muestra todas las personas de ese modelo concreto de la persona. Si hace clic en el botón de menú de la lista situada junto a cada una de las personas, ve administrar, cambiar el nombre y opciones de eliminación.  

![Agregar una nueva cara a la persona](./media/customize-face-model/manage-people.png)

### <a name="rename-a-person"></a>Cambiar el nombre de una persona

1. Para cambiar el nombre de una persona en el modelo de persona, haga clic en el botón de menú de lista y elija **cambiar el nombre** en el menú de lista.
1. Haga clic en el nombre actual de la persona y escriba el nuevo nombre.
1. Haga clic en el botón de comprobación y se cambiará la persona.

### <a name="delete-a-person"></a>Eliminar una persona

1. Para eliminar una persona de su modelo de persona, haga clic en el botón de menú de lista y elija **eliminar** en el menú de lista.
1. Una ventana emergente que indica que esta acción eliminará a la persona y que esta acción no se puede deshacer.
1. Haga clic en eliminar de nuevo y esta acción quitará la persona en el modelo de persona.

### <a name="manage-a-person"></a>Administrar una persona

Si hace clic en **administrar**, verá todas las caras que se esté entrenando el modelo de esta persona desde. Estas caras proceden de apariciones de esa persona en los vídeos que utilizan este modelo de persona o de imágenes que ha cargado manualmente. 

Puede agregar más caras a la persona haciendo clic en Agregar imágenes.

Puede usar el panel de administración para cambiar el nombre de la persona y eliminar a la persona desde el modelo de persona.

## <a name="use-a-person-model-to-index-a-video"></a>Utilice un modelo de la persona para indexar un vídeo

Puede usar un modelo de la persona a indexar el vídeo nuevo mediante la asignación del modelo de persona durante la carga del vídeo.

Para usar el modelo de persona en un nuevo vídeo, realice lo siguiente:

1. Haga clic en el **cargar** botón en la parte superior de la página.

    ![Cargar](./media/customize-face-model/upload.png)
1. Coloque el archivo de vídeo en el círculo o busque el archivo.
1. Haga clic en la flecha de opciones avanzadas.

    ![Cargar](./media/customize-face-model/upload2.png)
1. Haga clic en la lista desplegable y seleccione el modelo de la persona que ha creado.

    ![Cargar](./media/customize-face-model/upload3.png)
1. Haga clic en la opción de carga en la parte inferior de la página y el nuevo vídeo se indexarán mediante el modelo de la persona.

Si no especifica un modelo de persona durante la carga, Video Indexer indizará el vídeo con el modelo de persona predeterminado en su cuenta.

## <a name="use-a-person-model-to-reindex-a-video"></a>Usar un modelo de la persona a indizar un vídeo 

Para usar un modelo de la persona a indizar un vídeo en la colección, vaya a los vídeos de la cuenta en la página principal de Video Indexer y mantenga el puntero sobre el nombre del vídeo que desee volver a indizar.

Vea las opciones para editar, eliminar y volver a indizar el vídeo. 

1. Haga clic en la opción de volver a indizar el vídeo.

    ![Volver a indexar](./media/customize-face-model/reindex.png)

    Ahora puede seleccionar volver a indizar el vídeo con el modelo de persona.
1. Haga clic en la lista desplegable y seleccione el modelo de la persona que desea usar. 

    ![Volver a indexar](./media/customize-face-model/reindex2.png)

1. Haga clic en el **Reindex** botón y el vídeo se volver a indizar mediante el modelo de la persona.

Las nuevas modificaciones que realice en las caras detectadas y reconoce que simplemente volver a indexar el vídeo se guardará en el modelo de la persona que usó para indexar el vídeo.

## <a name="managing-people-in-your-videos"></a>Administración de las personas de sus vídeos

Puede administrar las caras detectadas y personas que se reconocen en los vídeos que indice mediante la edición y eliminación de caras.

Eliminación de una cara, se quita una cara específica de toda la información del vídeo.

Edición de una cara, cambia el nombre de una cara que se detectaron y posiblemente se reconocen en el vídeo. Cuando se edita una cara en el vídeo, ese nombre se guarda como una entrada de la persona en el modelo de la persona que asignó al vídeo durante la carga y la indización.

Si no asigna un modelo de la persona a la de vídeo durante la carga, la edición se guarda en el modelo de persona de su cuenta predeterminada.

### <a name="edit-a-face"></a>Edición de una cara


> [!NOTE]
> Si un modelo de persona tiene dos o más personas diferentes con el mismo nombre, no podrá etiquetar ese nombre de los vídeos que usan el modelo de esa persona. Sólo podrá realizar cambios en las personas que comparten el nombre en la ficha de personas de la página de personalización del modelo de contenido de Video Indexer. Por este motivo, se recomienda proporcionar nombres únicos a cada persona en el modelo de la persona.

1. Vaya al sitio Web de Video Indexer e inicie sesión.
1. Busque un vídeo que quiera ver y edítelo en su cuenta.
1. Para editar una cara en el vídeo, vaya a la pestaña de información y haga clic en el icono de lápiz en la esquina superior derecha de la ventana.

    ![Editar una cara del vídeo](./media/customize-face-model/edit-face.png)
1. Haga clic en cualquiera de las caras detectadas y cambie sus nombres "#X desconocido", o los que asignara previamente a la cara, por otros. 
1. Después de escribir el nuevo nombre, haga clic en el icono de verificación junto a él. Esto guarda el nuevo nombre y reconoce y los nombres de todas las apariciones de esta cara de los otros vídeos actuales y en el futuro vídeos cargados. Reconocimiento de la cara de los otros vídeos actuales puede tardar algún tiempo en aplicarse, tal como se trata de un proceso por lotes.

Si el nombre una cara con el nombre de una persona existente en el modelo de la persona que está usando el vídeo, se combinará las imágenes de la cara detectada en este vídeo de esa persona, con lo que ya existe en el modelo. Si el nombre una cara con un nombre completamente nuevo, esto creará una nueva entrada de la persona en el modelo de la persona que está usando el vídeo. 

    ![Edit a face in your video](./media/customize-face-model/edit-face2.png)

### <a name="delete-a-face"></a>Eliminación de un archivo

Para eliminar una cara detectada en el vídeo, vaya al panel de información y haga clic en el icono de lápiz en la esquina superior derecha del panel. Haga clic en la opción de eliminar debajo del nombre de la cara. Esta acción quitará esa cara detectada del vídeo. Cara de la persona todavía se detectarán en los otros vídeos en la que aparece, pero puede eliminar la cara de esos vídeos también después de que se han indizado. La persona que, si se ha tenido denominado, también seguirá existiendo en el modelo de la persona que se usó para indexar el vídeo que ha eliminado la cara a menos que elimine específicamente la persona del modelo de la persona.

![Eliminar una cara del vídeo](./media/customize-face-model/delete-face.png)


## <a name="next-steps"></a>Pasos siguientes

[Personalización del modelo de persona mediante las API](customize-person-model-with-api.md)
