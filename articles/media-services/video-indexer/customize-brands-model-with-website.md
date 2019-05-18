---
title: 'Uso del sitio web de Video Indexer para personalizar un modelo de marcas: Azure'
titlesuffix: Azure Media Services
description: En este artículo se muestra cómo personalizar un modelo de marcas con el sitio web de Video Indexer.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: anzaman
ms.openlocfilehash: 721fde63aeae8704761b3c21f489dcad77cb89e3
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799613"
---
# <a name="customize-a-brands-model-with-the-video-indexer-website"></a>Personalización de un modelo de marcas con el sitio web de Video Indexer

Video Indexer permite la detección de marcas por voz y texto visual durante la indexación y la reindexación de contenido de audio y vídeo. La característica de detección de marcas identifica menciones de productos, servicios y compañías sugeridas por la base de datos de marcas de Bing. Por ejemplo, si Microsoft se menciona en un contenido de audio o vídeo, o si se muestra en texto visual en un vídeo, Video Indexer lo detectará como una marca en el contenido. Un modelo de marcas personalizado le permite seleccionar si Video Indexer detectará o no marcas de la base de datos de marcas de Bing; excluirá determinadas marcas de la detección (mediante la creación de una lista de marcas no permitidas); e incluirá las marcas que deben ser parte del modelo que podrían no estar en la base de datos de marcas de Bing (mediante la creación de una lista de marcas permitidas).

Para información general detallada, consulte [Introducción](customize-brands-model-overview.md).

Puede utilizar el sitio web de Video Indexer para crear, usar y editar modelos de marcas personalizados detectados en un vídeo, como se describe en este tema. También puede usar la API, como se describe en [Personalización del modelo de marcas mediante las API](customize-brands-model-with-api.md).

## <a name="edit-the-settings-of-the-brands-model"></a>Edición de la configuración del modelo de marcas  

Tiene la opción de establecer si quiere que las marcas de la base de datos de marcas de Bing sean o no detectadas. Para ello, debe editar la configuración de su modelo de marcas.

1. Vaya al sitio web de [Video Indexer](https://www.videoindexer.ai/) e inicie sesión.
2. Para personalizar un modelo en su cuenta, haga clic en el botón **Content model customization** (Personalización del modelo de contenido) en la esquina superior derecha de la página.
 
   ![Personalización del modelo de contenido](./media/content-model-customization/content-model-customization.png) 
3. Para editar las marcas, seleccione la pestaña **Brands** (Marcas).

    ![Personalización del modelo de marcas](./media/customize-brand-model/customize-brand-model.png)
4. Marque la opción **Show brands suggested by Bing** (Mostrar marcas sugeridas por Bing) si quiere que Video Indexer incluya las marcas sugeridas por Bing. Deje desactivada la opción si no quiere que Video Indexer detecte las marcas sugeridas por Bing en su contenido. 

## <a name="include-brands-in-the-model"></a>Inclusión de marcas en el modelo

La sección **Include brands** (Incluir marcas) representa las marcas personalizadas que quiere que Video Indexer detecte aunque no las haya sugerido Bing.  

### <a name="add-a-brand"></a>Incorporación de una marca

1. Haga clic en "+ Add brand" (+Agregar marca).

    ![Personalización del modelo de marcas](./media/customize-brand-model/add-brand.png)

    Proporcione un nombre (obligatorio), la categoría (opcional), la descripción (opcional) y la dirección URL de referencia (opcional).
    El campo de categoría tiene como fin ayudarle a etiquetar sus marcas. Este campo se muestra como las *etiquetas* de la marca al usar las API de Video Indexer. Por ejemplo, la marca "Azure" se puede etiquetar o clasificar como "Nube".

    El campo de dirección URL de referencia puede ser cualquier sitio web de referencia para la marca, como un vínculo a su página de Wikipedia.
2. Haga clic en "Add brand" (Agregar marca) y verá que la marca se ha agregado a la lista **Include brands** (Incluir marcas).

### <a name="edit-a-brand"></a>Edición de una marca

1. Haga clic en el icono de lápiz junto a la marca que quiere editar.

    Puede actualizar la categoría, la descripción o la dirección URL de referencia de una marca. No puede cambiar el nombre de una marca porque los nombres de marcas son únicos. Si necesita cambiar el nombre de la marca, elimine la marca entera (consulte la sección siguiente) y cree una marca con el nuevo nombre.
2. Haga clic en el botón **Update** (Actualizar) para actualizar la marca con la nueva información.

### <a name="delete-a-brand"></a>Eliminación de una marca

1. Haga clic en el icono de papelera junto a la marca que quiere eliminar.
2. Haga clic en "Delete" (Eliminar) y la marca ya no aparecerá en la lista *Include brands* (Incluir marcas).

## <a name="exclude-brands-from-the-model"></a>Exclusión de marcas del modelo

La sección **Exclude brands** (Excluir marcas) representa las marcas que quiere que Video Indexer no detecte.

### <a name="add-a-brand"></a>Incorporación de una marca

1. Haga clic en "+ Add brand" (+Agregar marca).

    Proporcione un nombre (obligatorio) y una categoría (opcional).
2. Haga clic en "Agregar marca" y verá que la marca se ha agregado a la lista *Exclude brands* (Excluir marcas).

### <a name="edit-a-brand"></a>Edición de una marca

1. Haga clic en el icono de lápiz junto a la marca que quiere editar.

    Solo se puede actualizar la categoría de una marca. No puede cambiar el nombre de una marca porque los nombres de marcas son únicos. Si necesita cambiar el nombre de la marca, elimine la marca entera (consulte la sección siguiente) y cree una marca con el nuevo nombre.
2. Haga clic en el botón **Update** (Actualizar) para actualizar la marca con la nueva información.

### <a name="delete-a-brand"></a>Eliminación de una marca

1. Haga clic en el icono de papelera junto a la marca que quiere eliminar.
2. Haga clic en "Delete" (Eliminar) y la marca ya no aparecerá en la lista *Exclude brands* (Excluir marcas).

## <a name="next-steps"></a>Pasos siguientes

[Personalización del modelo de marcas mediante las API](customize-brands-model-with-api.md)
