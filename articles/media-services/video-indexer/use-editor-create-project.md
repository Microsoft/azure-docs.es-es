---
title: Utilice el editor de Video Indexer para crear proyectos
titlesuffix: Azure Media Services
description: En este tema se muestra cómo usar el editor de Video Indexer para crear proyectos.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 04/02/2019
ms.author: juliako
ms.openlocfilehash: 725e265bb7f8c98db19971c50e690974cb38a0d2
ms.sourcegitcommit: 17411cbf03c3fa3602e624e641099196769d718b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65518153"
---
# <a name="use-the-video-indexer-editor-to-create-projects"></a>Utilice el editor de Video Indexer para crear proyectos

Sitio Web de indizador vídeo, le permite usar detallados de los vídeos a: buscar el contenido multimedia derecho, busque las partes que le interesa y usar los resultados para crear un proyecto totalmente nuevo. Una vez creado, el proyecto puede representarse y descargado de Video Indexer y se puede usar en su propio de edición de las aplicaciones o flujos de trabajo de nivel inferiores.

Algunos escenarios donde puede encontrar esta característica útil son: 

* Creación de la película se resalta para remolques.
* Uso de antiguos clips de vídeos en conversiones de noticias.
* Creación de contenido más corto de medios sociales.

En este artículo se muestra cómo crear un proyecto desde cero y también cómo crear un proyecto desde un vídeo en su cuenta.

## <a name="create-new-project-and-manage-videos"></a>Crear nuevo proyecto y administración de vídeos

1. Vaya al sitio web de [Video Indexer](https://www.videoindexer.ai/) e inicie sesión.
1. Seleccione el **proyectos** ficha. Si ha creado proyectos antes, verá todos los demás proyectos aquí.
1. Haga clic en **crear nuevo proyecto**.  

    ![Nuevo proyecto](./media/video-indexer-view-edit/new-project.png)
1. Asigne un nombre al proyecto haciendo clic en el icono de lápiz. Reemplace el texto que dice "Proyecto sin título" por el nombre del proyecto y haga clic en la comprobación.

    ![Nuevo proyecto](./media/video-indexer-view-edit/new-project3.png)
    
### <a name="add-videos-to-the-project"></a>Agregar vídeos al proyecto

> [!NOTE]
> Actualmente, los proyectos solo pueden contener los vídeos que se indizan en el mismo idioma. Una vez que seleccione un vídeo en un idioma, no se puede agregar los vídeos en su cuenta de que se encuentran en un idioma diferente.

1. Agregar vídeos que desea trabajar en este proyecto seleccionando **agregar vídeos**.

    Verá todos los vídeos en su cuenta y un cuadro de búsqueda que dice "Buscar texto, las palabras clave o contenido visual". Para buscar vídeos que tienen una persona especificada, la etiqueta, la marca, la palabra clave o el aparición en la transcripción y el reconocimiento óptico de caracteres.
    
    Por ejemplo, en la imagen siguiente, estamos buscando vídeos que mencionan "GitHub".
    
    ![GitHub](./media/video-indexer-view-edit/github.png)

    Puede filtrar aún más los resultados seleccionando **filtrar resultados**. Puede filtrar para mostrar vídeos que tienen una determinada persona en ellos o para especificar que sólo desea ver los resultados de vídeo que son un determinado idioma o tiene un propietario específico. <br/> También puede especificar el ámbito de la consulta. Por ejemplo, si desea buscar "GitHub" en el reconocimiento óptico de caracteres, seleccione **texto Visual**.

    ![Filtro](./media/video-indexer-view-edit/visual-text.png)

    Puede agregar varios filtros a la consulta. Use la **+** / **-** botones para agregar o quitar filtros. Use **Borrar filtros** para quitar todos los filtros.
1. Para agregar vídeos, selecciónelos y, a continuación, seleccione **agregar**.
1. Ahora, verá todos los vídeos que eligió. Estos son los vídeos desde el que va a seleccionar los clips del proyecto.

    Puede reorganizar el orden de los vídeos arrastrando y colocando o seleccionando el botón de menú de lista y **Bajar** o **Subir**. En el menú de lista, también podrá quitar el vídeo de este proyecto. 

    ![Reorganizar](./media/video-indexer-view-edit/rearrange.png)
    
    Tiene la opción para agregar más vídeos a este proyecto en cualquier momento seleccionando **agregar vídeos**. También puede agregar varias apariciones del mismo vídeo al proyecto. Es posible que desee hacer esto si desea mostrar un clip de uno vídeo y, a continuación, en un clip de otro y, a continuación, otro clip desde el primer vídeo. 

### <a name="select-clips-to-use-in-your-project"></a>Seleccione clips para usar en el proyecto

Si hace clic en la flecha hacia abajo a la derecha de cada vídeo, se abrirá la información en el vídeo en función de las marcas de tiempo (clips de vídeo). 

1. Seleccione **ver información** para personalizar qué información desee ver y que no desea ver. 

    ![Ver conclusiones](./media/video-indexer-view-edit/insights.png)
1. Para crear consultas para clips específicos, use el cuadro de búsqueda que dice "Buscar en la transcripción, texto visual, las personas y las etiquetas".
1. Agregar filtros para especificar más detalles sobre qué escenas buscan seleccionando **opciones de filtro**.

    ![Opciones de filtro](./media/video-indexer-view-edit/filter-options.png)

    Por ejemplo, desea ver los clips que se menciona GitHub mientras Donovan Brown se encuentra en la pantalla. Para ello, deberá agregar un filtro "include" que tiene "People" como el tipo de información. A continuación, debe escribir "Donovan Brown" en el cuadro de búsqueda para el filtro.
    
    ![Incluir](./media/video-indexer-view-edit/include.png)
    
    Si desea que los clips que se menciona GitHub mientras está Donovan Brown _no_ en la pantalla, simplemente cambiaría el filtro "include" en un filtro de "exclude" mediante la lista desplegable. 

1. Agregar un clip al proyecto seleccionando el segmento que desea agregar. Puede anular la selección de esta galería haciendo clic en el segmento de nuevo.
    
    Agregar todos los segmentos de un vídeo, haga clic en la opción de menú de la lista situada junto a la selección y el vídeo **seleccione todos los segmentos**. 

    ![Agregar todos](./media/video-indexer-view-edit/add-all.png)

    Puede borrar toda la selección, seleccione Borrar selección.

> [!TIP]
> Como la selección y ordenación de los clips, puede obtener una vista previa del vídeo en el Reproductor en el lado derecho de la página. 

![Preview](./media/video-indexer-view-edit/preview.png)

Recuerde guardar el proyecto cuando se realizan cambios seleccionando **Guardar proyecto**. 

### <a name="render-and-download-the-project"></a>Representar y descargue el proyecto

> [!NOTE]
> Para las cuentas de pago de Video Indexer, representación de su proyecto tiene los costos de codificación. Cuentas de prueba de video Indexer se limitan a 5 horas de representación.

1. Cuando haya terminado, asegúrese de que se ha guardado el proyecto. Ahora puede representar este proyecto. Seleccione **representar y descargar**. 

    ![Save](./media/video-indexer-view-edit/save.png)

    Habrá un menú emergente que le indica que el indexador de vídeo representará un archivo y, a continuación, en que el vínculo de descarga se enviará a su correo electrónico. Seleccione continuar. 
    
    También verá una notificación de que el proyecto se representa encima de la página. Una vez que se hace que se va a representar, verá una nueva notificación de que el proyecto se ha procesado correctamente. Haga clic en la notificación para descargar el proyecto. Descargará el proyecto en formato mp4.

    ![Realizada la representación](./media/video-indexer-view-edit/rendering-done.png)

1. Puede tener acceso a proyectos guardados desde el **proyectos** ficha. 

    Si selecciona este proyecto, verá toda la información y la escala de tiempo de este proyecto. Si selecciona **editor vídeo**, aún puede realizar cambios en este proyecto. Ediciones incluyen agregar o quitar vídeos y clips o cambiar el nombre del proyecto.

    ![Editor de vídeo](./media/video-indexer-view-edit/video-editor.png)
     
## <a name="create-a-project-from-your-video"></a>Crear un proyecto desde el vídeo

Puede crear un nuevo proyecto directamente desde un vídeo en su cuenta. 

1. Vaya a la **biblioteca** ficha del sitio Web de Video Indexer.
1. Abra el vídeo que desea usar para crear el proyecto. En la página de escala de tiempo y insights, seleccione el **editor vídeo** botón.

    Esto le llevará a la misma página que usó para crear un nuevo proyecto. A diferencia del nuevo proyecto, verá los segmentos de insights con marca de tiempo del vídeo, que ha iniciado la edición anteriormente.

## <a name="see-also"></a>Vea también

[Introducción a Video Indexer](video-indexer-overview.md)

