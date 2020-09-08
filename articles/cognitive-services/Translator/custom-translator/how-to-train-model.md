---
title: 'Entrenamiento de un modelo: Custom Translator'
titleSuffix: Azure Cognitive Services
description: Entrenar un modelo es un paso importante al crear un modelo de traducción. El entrenamiento se produce según los documentos que seleccione para ello.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 824516a327d45feb5b6a084a113633bd3a486abe
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88508381"
---
# <a name="train-a-model"></a>Entrenamiento de un modelo

Entrenar un modelo es el primer paso y el más importante al crear un modelo de traducción, de lo contrario, el modelo no se puede generar. El entrenamiento se produce según los documentos que seleccione para ello. Al seleccionar documentos del tipo de documento "entrenamiento", tenga en cuenta el requisito mínimo de 10 000 oraciones paralelas. A medida que selecciona documentos, se muestra el número total de oraciones de entrenamiento que le servirán de guía. Este requisito no se aplica si selecciona solo documentos del tipo de documento diccionario para entrenar un modelo.

Para entrenar un modelo:

1. Seleccione el proyecto en el que desea crear un modelo.

2. La pestaña Datos del proyecto mostrará todos los documentos pertinentes para el par de idiomas del proyecto. Seleccione manualmente los documentos que desea usar para entrenar el modelo. Puede seleccionar documentos de aprendizaje, optimización y prueba desde esta pantalla. También basta con seleccionar el conjunto de aprendizaje y que Custom Translator cree automáticamente los conjuntos de optimización y prueba.

    - Nombre del documento: nombre que le asigna al documento

    - Emparejamiento: indica si este documento es un documento paralelo o monolingüe. En este momento, no se admiten los documentos monolingües para el entrenamiento.

    - Tipo de documento: puede ser de aprendizaje, optimización, prueba o diccionario.

    - Par de idiomas: esta opción muestra el idioma de origen y de destino para el proyecto.

    - Frases de origen: muestra el número de frases extraídas del archivo de origen.

    - Frases de destino: muestra el número de frases extraídas del archivo de destino.

    ![Entrenamiento de un modelo](media/how-to/how-to-train-model.png)

3. Haga clic en el botón "Create model" (Crear modelo).

4. En el cuadro de diálogo, especifique el nombre para el modelo. De forma predeterminada, aparece seleccionado "Train immediately" (Entrenar inmediatamente) para iniciar la canalización de entrenamiento al hacer clic en el botón "Create model" (Crear modelo). Puede seleccionar "Guardar como borrador" para crear los metadatos del modelo y poner a este en un estado de borrador, pero el entrenamiento del modelo no se iniciará. Más adelante, tendrá que seleccionar manualmente los modelos en el estado de borrador para entrenarlos.

5. Haga clic en el botón "Create model" (Crear modelo).

    ![Cuadro de diálogo Entrenar modelo](media/how-to/how-to-train-model-2.png)

6. Traductor personalizado enviará el entrenamiento y mostrará el estado del mismo en la pestaña de modelos.

    ![Página Entrenar modelo](media/how-to/how-to-train-model-3.png)

>[!Note]
>El traductor personalizado es compatible con 10 modelos de aprendizaje de un área de trabajo en cualquier momento dado.

## <a name="modify-a-model-name"></a>Modificación del nombre de un modelo

Puede modificar el nombre de un modelo en la página Detalles del modelo.

1. En la página de proyectos, haga clic en el nombre del proyecto en el que existe el modelo.
2. Haga clic en la pestaña del modelo.
3. Haga clic en el nombre del modelo para ver sus detalles.
4. Haga clic en el icono de lápiz.

    ![Editar modelo](media/how-to/how-to-edit-model.png)

5. En el cuadro de diálogo, cambie el nombre del modelo y asígnele un nombre descriptivo.

    ![Cuadro de diálogo Editar modelo](media/how-to/how-to-edit-model-dialog.png)

6. Haga clic en Guardar.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [ver los detalles del modelo](how-to-view-model-details.md).
