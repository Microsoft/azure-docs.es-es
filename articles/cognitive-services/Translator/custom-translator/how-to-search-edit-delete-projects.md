---
title: 'Cómo buscar, editar y eliminar proyectos: Custom Translator'
titleSuffix: Azure Cognitive Services
description: Custom Translator proporciona varias formas de administrar los proyectos de manera eficaz. Puede crear varios proyectos, realizar búsquedas según los criterios que elija y editar sus proyectos. También es posible eliminar proyectos de Custom Translator.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 4c48a8a18a3ad0487b4c3840847c7f5edbb6d29f
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88510383"
---
# <a name="search-edit-and-delete-projects"></a>Búsqueda, edición y eliminación de proyectos

Custom Translator proporciona varias formas de administrar los proyectos de manera eficaz. Puede crear muchos proyectos, realizar búsquedas según los criterios que elija y editar sus proyectos. También es posible eliminar proyectos de Custom Translator.  

## <a name="search-and-filter-projects"></a>Búsqueda y filtrado de proyectos

La herramienta de filtro le permite buscar proyectos mediante condiciones de filtro diferentes. Se pueden utilizar como filtros el nombre del proyecto, el estado, los idiomas de origen y de destino y la categoría del proyecto.

1. Haga clic en el botón de filtro.

    ![Búsqueda de proyectos](media/how-to/how-to-search-project.png)

2. Puede filtrar por cualquiera de los siguientes campos o usarlos todos: nombre de proyecto, idioma de origen, idioma de destino, categoría y disponibilidad de proyectos.

3. Haga clic en Aplicar.

    ![Opciones de filtrado para la búsqueda de proyectos](media/how-to/how-to-search-project-filters.png)

4. Puede borrar el filtro para ver todos los proyectos si pulsa "Borrar".

## <a name="edit-a-project"></a>Edición de un proyecto

El traductor personalizado le ofrece la posibilidad de editar el nombre y la descripción de un proyecto. Otros metadatos del proyecto, como la categoría o los idiomas de origen y de destino no están disponibles para su edición. Los pasos siguientes describen cómo editar un proyecto.

1. Haga clic en el icono de lápiz que aparece cuando mantiene el mouse sobre un proyecto.

    ![Editar un proyecto](media/how-to/how-to-edit-project.png)

2. En el cuadro de diálogo, puede modificar el nombre del proyecto, la descripción del proyecto, la descripción de la categoría y la etiqueta de proyecto si no se implementa ningún modelo. No se puede modificar la categoría o el par de idiomas una vez creado el proyecto.

    ![Cuadro de diálogo Editar un proyecto](media/how-to/how-to-edit-project-dialog.png)

3. Haga clic en el botón "Guardar".

## <a name="delete-a-project"></a>Eliminación de un proyecto

Puede eliminar un proyecto cuando ya no lo necesite. Haga que el proyecto no tenga modelos en estado activo, como implementado, entrenamiento enviado, procesamiento de datos, implementación, etc.; de lo contrario, se producirá un error en la operación de eliminación. Los pasos siguientes describen cómo eliminar un proyecto.

1. Mantenga el mouse sobre cualquier registro de proyecto y haga clic en el icono de papelera.

   ![Eliminar proyecto](media/how-to/how-to-delete-project.png)

2. Confirme la eliminación. Al eliminar un proyecto, se eliminarán todos los modelos que se crearon dentro de ese proyecto. La eliminación de un proyecto no afectará los documentos.

   ![Cuadro de diálogo para confirmar la eliminación](media/how-to/how-to-delete-project-confirm.png)

## <a name="next-steps"></a>Pasos siguientes

- [Cargar documentos](how-to-upload-document.md) para comenzar a crear el modelo de traducción personalizada.
