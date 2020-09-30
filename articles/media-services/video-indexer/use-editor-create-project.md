---
title: Utilizar el editor de Video Indexer para crear proyectos
titleSuffix: Azure Media Services
description: En este tema se muestra cómo usar el editor de Video Indexer para crear proyectos.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 04/02/2019
ms.author: juliako
ms.openlocfilehash: 5eaf17281fef32d682a2dac17b379faf7ec23a53
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90970092"
---
# <a name="use-the-video-indexer-editor-to-create-projects"></a>Utilizar el editor de Video Indexer para crear proyectos

El sitio web de Video Indexer le permite usar información pormenorizada de los vídeos para buscar el contenido multimedia adecuado, localizar las partes que le interesen y usar los resultados para crear un proyecto totalmente nuevo. Una vez creado, puede representar y descargar el proyecto desde Video Indexer y usarlo en sus propias aplicaciones de edición o en sus flujos de trabajo descendentes.

Esta característica puede resultarle útil para: 

* Crear destacados de películas en clips finales.
* Usar clips de vídeos antiguos en transmisiones de noticias.
* Crear contenido más breve para las redes sociales.

En este artículo se muestra cómo crear un proyecto desde cero y también cómo crear un proyecto a partir de un vídeo de su cuenta.

## <a name="create-new-project-and-manage-videos"></a>Crear un nuevo proyecto y administrar vídeos

1. Vaya al sitio web de [Video Indexer](https://www.videoindexer.ai/) e inicie sesión.
1. Seleccione la pestaña **Proyectos**. Si ha creado proyectos anteriormente, verá todos los demás proyectos aquí.
1. Haga clic en **Crear proyecto**.  

    ![Nuevo proyecto](./media/video-indexer-view-edit/new-project.png)
1. Asigne un nombre al proyecto haciendo clic en el icono de lápiz. Reemplace el texto “Untitled project” (Proyecto sin título) por el nombre del proyecto y haga clic en la marca de verificación.

    ![Nuevo proyecto](./media/video-indexer-view-edit/new-project3.png)
    
### <a name="add-videos-to-the-project"></a>Agregar vídeos al proyecto

> [!NOTE]
> Actualmente, los proyectos solo pueden contener vídeos indexados en el mismo idioma. Cuando seleccione un vídeo en un idioma, no podrá agregar vídeos de su cuenta que estén en otro idioma.

1. Agregue los vídeos con los que quiera trabajar en este proyecto seleccionando **Agregar vídeos**.

    Verá todos los vídeos de su cuenta y un cuadro de búsqueda con el texto “Buscar texto, palabras clave o contenido visual”, que le permite buscar vídeos que contengan una persona, etiqueta, marca, palabra clave o instancia concretas en la transcripción y el reconocimiento de caracteres ópticos.
    
    Por ejemplo, en la imagen siguiente estamos buscando vídeos en los que se mencione “GitHub”.
    
    ![Captura de pantalla que muestra la búsqueda de vídeos que mencionan GitHub con dos resultados.](./media/video-indexer-view-edit/github.png)

    Puede filtrar aún más los resultados seleccionando **Filter results** (Filtrar resultados). Puede filtrar los vídeos para que se muestren los que contengan una determinada persona o para especificar que solo quiere ver los resultados de vídeos que estén en un idioma determinado o que sean de un propietario concreto. <br/> También puede especificar el ámbito de la consulta. Por ejemplo, si desea buscar “GitHub” en el reconocimiento óptico de caracteres, seleccione **Solo texto visual**.

    ![Filter](./media/video-indexer-view-edit/visual-text.png)

    Puede agregar varios filtros a la consulta. Use los botones **+** / **-** para agregar o quitar filtros. Use **Clear filters** (Borrar filtros) para quitar todos los filtros.
1. Para agregar vídeos, selecciónelos y después seleccione **Add** (Agregar).
1. Ahora verá todos los vídeos que ha elegido. Estos son los vídeos de los que seleccionará clips para el proyecto.

    Puede cambiar el orden de los vídeos arrastrándolos y soltándolos o seleccionando el botón de menú de la lista y seleccionando **Move down** (Bajar) o **Move up** (Subir). Desde el menú de lista, también podrá quitar el vídeo de este proyecto. 

    ![Captura de pantalla que muestra Video Indexer con un menú contextual para uno de los vídeos que se van a quitar, borrar la selección o bajar.](./media/video-indexer-view-edit/rearrange.png)
    
    Puede agregar más vídeos a este proyecto en cualquier momento seleccionando **Add videos** (Agregar vídeos). También puede agregar al proyecto varias instancias del mismo vídeo. Esto podría interesarle si quisiera mostrar un clip de un vídeo, después un clip de otro vídeo y, por último, otro clip del primero. 

### <a name="select-clips-to-use-in-your-project"></a>Seleccionar clips para usar en el proyecto

Si hace clic en la flecha hacia abajo de la derecha de cada vídeo, se abrirá información del vídeo basada en las marcas de tiempo (clips de vídeo). 

1. Seleccione **View Insights** (Ver información) para personalizar qué información quiere ver y cuál no. 

    ![Ver información](./media/video-indexer-view-edit/insights.png)
1. Para crear consultas para clips específicos, use el cuadro de búsqueda en el que se lee “Search in transcript, visual text, people, and labels” (Buscar transcripciones, texto visual, personas y etiquetas).
1. Puede agregar filtros para especificar con más detalle qué escenas está buscando seleccionando **Filter options** (Opciones de filtro).

    ![Opciones de filtro](./media/video-indexer-view-edit/filter-options.png)

    Por ejemplo, si quiere ver clips en los que se menciona GitHub mientras Donovan Brown está en pantalla, agregue el filtro “include” (incluir) que tenga “People” (Personas) como tipo de información. Después deberá escribir “Donovan Brown” en el cuadro de búsqueda del filtro.
    
    ![Captura de pantalla que muestra Video Indexer con personas seleccionadas para un filtro de inclusión.](./media/video-indexer-view-edit/include.png)
    
    Si quiere ver clips en los que se menciona GitHub mientras Donovan Brown _no_ está en pantalla, cambie el filtro “include” (incluir) por el filtro “exclude” (excluir) usando el menú desplegable. 

1. Agregue un clip al proyecto seleccionando el segmento que quiera agregar. Puede anular la selección de este clip volviendo a hacer clic en el segmento.
    
    Para agregar todos los segmentos de un vídeo, haga clic en la opción de menú de la lista situada junto al vídeo y seleccione **Select all segments** (Seleccionar todos los segmentos). 

    ![Agregar todo](./media/video-indexer-view-edit/add-all.png)

    Puede borrar toda su selección con la opción “Clear selection” (Borrar selección).

> [!TIP]
> A medida que seleccione y ordene los clips, puede obtener una vista previa del vídeo en el reproductor situado en el lado derecho de la página. 

![Captura de pantalla que muestra Video Indexer con la vista previa de un vídeo en el lado derecho de la ventana.](./media/video-indexer-view-edit/preview.png)

Recuerde que, cuando haga cambios, deberá guardar el proyecto seleccionando **Save project** (Guardar proyecto). 

### <a name="render-and-download-the-project"></a>Renderizar y descargar el proyecto

> [!NOTE]
> En las cuentas de pago de Video Indexer, la renderización del proyecto tiene costes de codificación. Con las cuentas de prueba de Video Indexer puede renderizar un máximo de cinco horas.

1. Cuando haya terminado, guarde el proyecto. Ahora puede renderizar este proyecto. Seleccione **Render and Download** (Renderizar y descargar). 

    ![Captura de pantalla que muestra Video Indexer con la opción para presentar y descargar el proyecto.](./media/video-indexer-view-edit/save.png)

    Verá un menú emergente en el que se le indica que Video  Indexer renderizará un archivo y después le enviará el vínculo de descarga al correo. Seleccione Proceed (Continuar). 
    
    También verá una notificación en la que se le indica que el proyecto se está renderizando en la parte superior de la página. Cuando haya finalizado la renderización, verá una nueva notificación en la que se le indica que el proyecto se ha renderizado correctamente. Haga clic en la notificación para descargar el proyecto en formato mp4.

    ![Renderización finalizada](./media/video-indexer-view-edit/rendering-done.png)

1. Puede acceder a los proyectos guardados desde la pestaña **Proyectos**. 

    Si selecciona este proyecto, verá toda la información relacionada y su escala de tiempo. Si selecciona el **editor de vídeo**, puede seguir editando el proyecto. Durante la edición, puede agregar o quitar vídeos y clips, o cambiar el nombre del proyecto.

    ![Editor de vídeo](./media/video-indexer-view-edit/video-editor.png)
     
## <a name="create-a-project-from-your-video"></a>Crear un proyecto a partir de un vídeo

Puede crear un nuevo proyecto directamente desde un vídeo de su cuenta. 

1. Vaya a la pestaña **Biblioteca** del sitio web de Video Indexer.
1. Abra el vídeo que quiera usar para crear el proyecto. En la página de escala de tiempo y conclusiones, seleccione el botón **Editar vídeo**.

    Se le dirigirá a la misma página que usó para crear un nuevo proyecto. A diferencia del nuevo proyecto, verá los segmentos de información con marca de tiempo del vídeo que ha empezado a editar.

## <a name="see-also"></a>Consulte también

[Introducción a Video Indexer](video-indexer-overview.md)

