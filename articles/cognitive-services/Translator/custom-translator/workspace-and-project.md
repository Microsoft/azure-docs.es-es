---
title: ¿Qué es un área de trabajo y un proyecto? - Custom Translator
titleSuffix: Azure Cognitive Services
description: En este artículo se explican las diferencias entre un área de trabajo y un proyecto, así como las categorías y etiquetas de proyecto para el servicio de Traductor personalizado.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 41586b2ec96d30fd583cbc9a746493ed9206548b
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88510749"
---
# <a name="what-is-a-custom-translator-workspace"></a>¿Qué es un área de trabajo de Custom Translator?

Un área de trabajo es un área para crear y compilar el sistema de traducción personalizada. Un área de trabajo puede contener varios proyectos, modelos y documentos. Todo el trabajo que realice en Custom Translator se encuentra dentro de un área de trabajo específica.

Las áreas de trabajo son privadas y solo pueden acceder a ellas usted mismo y las personas a las que invite. Las personas a las que no invite no podrán acceder al contenido del área de trabajo. Puede invitar a tantas personas como quiera al área de trabajo y modificar o quitar el acceso en cualquier momento. También puede crear una nueva área de trabajo. De forma predeterminada, un área de trabajo no contendrá ninguno de los proyectos o documentos que están dentro de las otras áreas de trabajo.

## <a name="what-is-a-custom-translator-project"></a>¿Qué es un proyecto de Custom Translator?

Un proyecto es un contenedor para un modelo, documentos y pruebas. Cada proyecto incluye automáticamente todos los documentos que se cargan en esa área de trabajo con el par de idiomas correcto. Por ejemplo, si tiene proyectos en inglés y en español, y un proyecto de inglés a español, se incluirán los mismos documentos en todos los proyectos. Cada proyecto tiene un id. de categoría asociado que se usa en las consultas a la [API V3](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-translate?tabs=curl) para recibir las traducciones. El parámetro id. de categoría se utiliza para obtener las traducciones de un sistema personalizado creado con Custom Translator.

## <a name="project-categories"></a>Categorías de proyecto

La categoría identifica el dominio (el área de terminología y estilo que quiere usar) para el proyecto. Elija la que mejor se ajuste a los documentos. En algunos casos, su elección de categoría influye directamente en el comportamiento de Custom Translator.

Tenemos dos conjuntos de modelos de línea base. Son General y Tecnología. Si la categoría **Tecnología** está seleccionada, se utilizarán los modelos de línea de base técnicos. Para cualquier otra selección de categoría, se utilizan los modelos de línea de base generales. El modelo de línea de base tecnológico se desempeña bien en el ámbito de la tecnología, pero muestra una calidad inferior, si las frases que se utilizan para la traducción no se encuentran dentro del dominio tecnológico. Se recomienda que los clientes seleccionen la categoría Tecnología solo si las frases se encuentran estrictamente dentro del dominio tecnológico.

En la misma área de trabajo, puede crear proyectos para el mismo par de idiomas en distintas categorías. Custom Translator no permite la creación de un proyecto duplicado con el mismo par de idiomas y categoría. Si aplica una etiqueta al proyecto, podrá evitar esta restricción. No use las etiquetas a menos que esté creando sistemas de traducción para varios clientes, ya que la adición de una etiqueta única al proyecto se verá reflejada en el id. de categoría del proyecto.

## <a name="project-labels"></a>Etiquetas de proyecto

Custom Translator le permite asignar una etiqueta de proyecto al proyecto. La etiqueta de proyecto distingue entre varios proyectos con el mismo par de idiomas y categoría. Como práctica recomendada, evite usar etiquetas de proyecto a menos que sea necesario.

La etiqueta de proyecto se usa como parte del id. de categoría. Si la etiqueta del proyecto se deja sin establecer o se establece de forma idéntica entre proyectos, los proyectos con la misma categoría y *diferentes* pares de idiomas compartirán el mismo id. de categoría. Este enfoque resulta ventajoso porque permite que usted o su cliente cambien entre idiomas cuando usen la Text Translator API sin preocuparse por que un id. de categoría sea único para cada proyecto.

Por ejemplo, si quisiera permitir traducciones en el dominio Tecnología del inglés al francés y del francés al inglés, crearía dos proyectos: uno para inglés -\> francés y otro para francés -\> inglés. Especificaría la misma categoría para ambos (Tecnología) y dejaría en blanco la etiqueta del proyecto. El id. de categoría para ambos proyectos coincidiría, por lo que podría consultar la API para las traducciones al inglés y al francés sin tener que modificar el valor del id. de categoría.

Si es un proveedor de servicios de lenguaje y quiere prestar servicios a varios clientes con diferentes modelos que conserven la misma categoría y par de idiomas, utilizar una etiqueta de proyecto para diferenciar clientes sería una decisión adecuada.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre [entrenamientos y modelos](training-and-model.md) para saber cómo crear un modelo de traducción de forma eficiente.
