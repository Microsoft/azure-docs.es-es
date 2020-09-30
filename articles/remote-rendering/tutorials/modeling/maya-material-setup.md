---
title: Configuración de materiales de representación con representación física en Maya
description: En este artículo se explica cómo configurar materiales con representación física en Maya y exportarlos en formato FBX.
author: FlorianBorn71
ms.author: flborn
ms.date: 06/16/2020
ms.topic: tutorial
ms.openlocfilehash: 43c11575be084354e4c9287b44b5847d4d086aff
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90084403"
---
# <a name="tutorial-set-up-physically-based-rendering-materials-in-maya"></a>Tutorial: Configuración de materiales de representación con representación física en Maya

## <a name="overview"></a>Información general
En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Asignar materiales con iluminación avanzada a los objetos de la escena.
> * Administrar la creación de instancias de objetos y materiales.
> * Exportar una escena a formato FBX y seleccionar las opciones importantes.

La creación de [materiales de representación con representación física (PBR)](../../overview/features/pbr-materials.md) en Maya es una tarea relativamente sencilla. Es similar de muchas maneras a la configuración de PBR en otras aplicaciones de creación de contenido como 3DS Max. Este tutorial es una guía para la configuración básica del sombreador de PBR y la exportación a FBX de proyectos de Azure Remote Rendering. 

La escena de ejemplo de este tutorial contiene varios objetos de cuadro poligonal. Se les han asignado distintos materiales, como madera, metal, metal pintado, plástico y caucho. En términos generales, cada material contiene todas o la mayoría de las texturas siguientes:

* **Albedo**, que es el mapa de colores del material y también se llama **Difuso** o **Color de base**.
* **Metálico**, que determina si un material es metálico y qué partes son metálicas. 
* **Rugosidad**, que determina cómo de rugosa o suave es una superficie y afecta a la nitidez o al desenfoque de los reflejos y los resaltados en una superficie.
* **Normal**, que agrega detalles a una superficie sin necesidad de agregar más polígonos. Algunos ejemplos de detalle podrían ser las picaduras y las abolladuras en una superficie de metal o el grano en la madera.
* **Oclusión de ambiente**, que se usa para agregar matices suaves y sombras de contacto a un modelo. Se trata de un mapa de escala de grises que indica qué áreas de un modelo reciben una iluminación completa (blanco) o una sombra completa (negro). 

## <a name="prerequisites"></a>Requisitos previos
* Autodesk Maya 2017 o posterior

## <a name="set-up-materials-in-the-scene"></a>Configuración de los materiales de la escena
Aquí se muestra cómo configurar un material de PBR en Maya.

Como verá en la escena de ejemplo, hemos creado varios objetos de cuadro. Cada objeto representa un tipo de material diferente. Tal y como se muestra en la imagen, a cada uno de estos objetos se le ha asignado un nombre adecuado.

Azure Remote Rendering usa metros para las medias y la dirección hacia arriba es el eje Y. Antes de empezar a crear recursos, se recomienda establecer las unidades de la escena de Maya en metros. Para la exportación, establezca las unidades en metros en la configuración de exportación de FBX.

> [!TIP]
> Asigne a los recursos del modelo los nombres adecuados en función del tipo de material o la parte correspondiente. Los nombres descriptivos facilitan la exploración de escenas con gran cantidad de objetos.

![Nombres de objetos](media/object-names.jpg)

Después de crear o adquirir algunas texturas, también puede crear texturas únicas. Puede usar aplicaciones de creación de texturas como Quixel Suite, PhotoShop o Substance Suite u obtener texturas de mosaico genéricas de otros orígenes.

Para aplicar texturas al modelo:

1. En la ventanilla de la escena, seleccione el modelo o la geometría y haga clic con el botón derecho en ellos. En el menú que aparece, seleccione **Assign New Material** (Asignar nuevo material).
1. En el cuadro de diálogo **Assign New Material** (Asignar nuevo material), vaya a **Maya** > **Stingray PBS** (Maya>PBS Stingray). Esta acción asigna un material de PBR al modelo. 

En Maya 2020, hay disponibles varios sombreadores de PBR diferentes. Entre ellos se incluyen **Maya Standard Surface** (Superficie estándar de Maya), **Arnold Standard Surface** (Superficie estándar Arnold), and **Stingray PBR** (PBR Stingray). El **sombreador de superficies estándar de Maya** aún no se puede exportar mediante el complemento de FBX 2020. El **sombreador de superficies estándar Arnold** se puede exportar con archivos FBX. En la mayoría de los demás aspectos, es idéntico al **sombreador de superficies estándar de Maya**. Es análogo al **material físico** de 3D Studio Max.

El **sombreador de PBR Stingray** es compatible con muchas otras aplicaciones y coincide en gran medida con los requisitos de Azure Remote Rendering. Se admite desde Maya 2017. Cuando este tipo de material se visualiza en la ventanilla, es similar a lo que se visualiza más adelante en Azure Remote Rendering.

![Material Stingray](media/stingray-material.jpg)

Una vez que el material se ha asignado al recurso y se le ha asignado un nombre adecuado, ahora puede asignar las diversas texturas. En las siguientes imágenes se muestra dónde encaja cada tipo de textura en el material de PBR. El material de PBR Stingray permite seleccionar los atributos que se pueden activar. Antes de conectar los mapas de texturas, debe activar los atributos correspondientes.

![Configuración de los materiales](media/material-setup.jpg)

Asigne a los materiales un nombre adecuado, teniendo en cuenta su uso o tipo. A un material que se utiliza en una única parte se le puede asignar un nombre relativo a dicha parte. A un material que se utiliza en un amplio número de áreas se le puede asignar un nombre en función de sus propiedades o de su tipo.

Asigne las texturas tal como se muestra en la imagen.

![Configuración de las texturas](media/texture-setup.jpg)

Una vez que los materiales de PBR se han creado y configurado, puede pensar en la [creación de instancias de los objetos](../../how-tos/conversion/configure-model-conversion.md#instancing) de la escena. La creación de instancias de objetos similares de la escena, como tuercas, pernos, tornillos y arandelas, produce un ahorro significativo en el tamaño de archivo. Las instancias de un objeto maestro pueden tener su propia escala, rotación y transformaciones, por lo que se pueden colocar donde sea necesario en la escena. 

En Maya, el proceso de creación de instancias es sencillo.

1. En el menú **Edit** (Editar), vaya a **Duplicate Special** (Duplicado especial) para abrir las opciones.
1. En el cuadro de diálogo **Duplicate Special Options** (Opciones de duplicado especial), seleccione la opción **Instance** (Instancia) para el campo **Geometry type** (Tipo de geometría). 
1. Seleccione **Duplicate Special** (Duplicado especial).

   ![Instanciación](media/instancing.jpg)

Esta acción crea una instancia del objeto. Puede moverla, girarla o escalarla independientemente de su elemento primario y de otras instancias de ese elemento primario. 

Los cambios que realice en una instancia mientras está en modo de componente se transmiten a todas las instancias del objeto. Por ejemplo, podría trabajar con los componentes de un objeto de instancia, como los vértices y las caras del polígono. Asegúrese de que desea que los cambios realizados afecten a todas estas instancias. 

En la escena de ejemplo, se ha creado una instancia de cada objeto de cuadro individual. Esta acción tendrá importancia cuando se exporte la escena a formato FBX.

![Información general de la escena](media/scene-overview.jpg)

> [!TIP]
> Cree instancias en la escena a medida que avance. Es muy difícil reemplazar las copias por objetos con instancias más adelante. 

## <a name="fbx-export-process"></a>Proceso de exportación de FBX

Vamos a pasar a la exportación de FBX de la escena o de los recursos de la escena. Al exportar recursos, tiene sentido seleccionar solo los objetos o recursos de la escena que desea exportar. Por ejemplo, podría tener 100 objetos en una escena. Si solo quiere usar 30 de ellos, no tiene sentido la exportación de toda la escena. 

Para realizar la selección:

1. Vaya a **File** > **Export Selection** (Archivo>Selección de exportación) para abrir el cuadro de diálogo **Export Selection** (Selección de exportación).
1. En el cuadro de texto **Files of type** (Archivos de tipo), seleccione **FBX export** (Exportación de FBX) para mostrar la configuración de exportación de FBX. La configuración principal de la exportación de FBX se resalta en rojo en la imagen.

   ![Exportación de FBX](media/FBX-exporting.jpg)

En función de sus requisitos, por ejemplo, puede que desee enviar un recurso a un cliente. Quizás no quiera enviar un gran número de archivos de textura con el recurso. Puede optar por insertar las texturas dentro del archivo FBX exportado. Esta opción significa que solo tiene un archivo para empaquetar, pero aumentará significativamente el tamaño del recurso de FBX. Para habilitar la opción para insertar texturas, seleccione la opción **Embed Media** (Insertar elementos multimedia) tal como se muestra.

> [!TIP]
> En este ejemplo, se asigna al archivo un nombre que refleje esta condición. Este estilo de nomenclatura es una buena forma de realizar un seguimiento de los recursos. 

Después de establecer la configuración de la exportación, seleccione **Export Selection** (Selección de exportación) en la parte inferior derecha.

![Inserción de elementos multimedia](media/embedding-media.jpg)

## <a name="conclusion"></a>Conclusión

En general, este tipo de materiales es más realista, ya que se basa en la física de la luz del mundo real. Crea un efecto envolvente adicional para que la escena parezca ser del mundo real.

## <a name="next-steps"></a>Pasos siguientes

Ahora ya sabe cómo configurar materiales con iluminación avanzada para los objetos de una escena. También sabe cómo exportar los objetos al formato FBX compatible con Azure Remote Rendering. El siguiente paso consiste en convertir el archivo FBX y visualizarlo en Azure Remote Rendering.

> [!div class="nextstepaction"]
> [Inicio rápido: Conversión de un modelo para su representación](../../quickstarts\convert-model.md)