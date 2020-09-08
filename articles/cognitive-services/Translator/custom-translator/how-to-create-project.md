---
title: 'Creación de un proyecto: Custom Translator'
titleSuffix: Azure Cognitive Services
description: En este artículo se explica cómo crear y administrar un proyecto en el Traductor personalizado de Azure Cognitive Services.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 3043ec0600f802a409387f0114aae5f56151da16
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88511000"
---
# <a name="create-a-project"></a>Crear un proyecto

Un proyecto es un contenedor para un modelo, documentos y pruebas. Cada proyecto incluye automáticamente todos los documentos que se cargan en esa área de trabajo con el par de idiomas correcto.

Crear un proyecto es el primer paso para crear un modelo.

## <a name="create-a-project"></a>Crear un proyecto:

1.  En el portal de [Custom Translator](https://portal.customtranslator.azure.ai), haga clic en Crear proyecto.

    ![Crear proyecto](media/how-to/how-to-create-project.png)

2.  Escriba la siguiente información sobre el proyecto en el cuadro de diálogo:

    a.  Project name (Nombre del proyecto) (obligatorio): asigne al proyecto un nombre único y descriptivo. No es necesario mencionar los idiomas en el título.

    b.  Descripción: un breve resumen sobre el proyecto. Esta descripción no influye en el comportamiento de Custom Translator ni en el sistema personalizado resultante, pero puede ayudar a distinguir entre distintos proyectos.

    c.  Language pair (Par de idiomas) (obligatorio): seleccione los idiomas de origen y destino de la traducción.

    d.  Category (Categoría) (obligatorio): seleccione la categoría más adecuada para su proyecto. La categoría describe la terminología y el estilo de los documentos que se van a traducir.

    e.  Category description (Descripción de la categoría): use este campo para describir mejor el sector o la industria en concreto en los que está trabajando. Por ejemplo, si la categoría es "medicina", podría agregar un documento concreto, como cirugía o pediatría. Esta descripción no influye en el comportamiento de Custom Translator ni en el sistema personalizado resultante.

    f.  Project label (Etiqueta del proyecto): la [etiqueta de proyecto](workspace-and-project.md#project-labels) distingue entre proyectos con el mismo par de idiomas y categoría. Como práctica recomendada, use una etiqueta *solo* si piensa compilar varios proyectos para el mismo par de idiomas y la misma categoría y quiere tener acceso a estos proyectos con un id. de categoría distinto. No use este campo si está creando sistemas para una sola categoría. Las etiquetas de proyecto no son obligatorias y no ayudan a distinguir entre pares de idiomas. Puede usar la misma etiqueta para varios proyectos.

    ![Cuadro de diálogo Crear proyecto](media/how-to/how-to-create-project-dialog.png)

3.  Haga clic en Crear.

## <a name="view-project-details"></a>Ver los detalles del proyecto

La página principal de Custom Translator muestra los primeros 10 proyectos en el área de trabajo. Muestra el nombre del proyecto, el par de idiomas, la categoría, el estado y la puntuación BLEU.

Después de seleccionar un proyecto, verá lo siguiente en la página del proyecto:

- CategoryID (Identificador de categoría): se crea al concatenar el identificador de área de trabajo, la etiqueta de proyecto y el código de categoría. Utilice el id. de categoría con Translator Text API para obtener traducciones personalizadas. Para copiar, haga clic en el icono de copia.

- Botón Train (Entrenar): use este botón para iniciar el [entrenamiento de un modelo](how-to-train-model.md).

- Botón Add documents (Agregar documentos): use este botón para [cargar documentos](how-to-upload-document.md).

- Botón Filter documents (Filtrar documentos): use este botón para filtrar y buscar documentos específicos.

    ![Ver los detalles del proyecto](media/how-to/how-to-view-project.png)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre [cómo buscar, editar y eliminar el proyecto](how-to-search-edit-delete-projects.md).
- Obtenga información sobre [cómo cargar un documento](how-to-upload-document.md) para generar modelos de traducción.
