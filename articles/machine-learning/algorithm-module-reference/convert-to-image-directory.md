---
title: Conversión al directorio de imagen
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo Conversión al directorio de imagen para convertir el conjunto de datos al formato de directorio de imagen.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 05/26/2020
ms.openlocfilehash: 677cf60ff3e614fd1486445786154fbf026b7cd9
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90898695"
---
# <a name="convert-to-image-directory"></a>Conversión al directorio de imagen

En este artículo se describe cómo usar el módulo Conversión al directorio de imagen para ayudar a convertir el conjunto de datos de imagen al tipo de datos "Directorio de imagen", que es el formato de datos estandarizado en tareas relacionadas con imágenes como la clasificación de imágenes en el diseñador de Azure Machine Learning.

## <a name="how-to-use-convert-to-image-directory"></a>Cómo usar Conversión al directorio de imagen  

1.  Agregue el módulo **Conversión al directorio de imagen** al experimento. Puede encontrar este módulo en la categoría "Computer Vision/Image Data Transformation" de la lista de módulos. 

2.  [Registre un conjunto de datos de imagen](https://docs.microsoft.com/azure/machine-learning/how-to-create-register-datasets) y conéctelo al puerto de entrada del módulo. Asegúrese de que haya una imagen en el conjunto de datos de entrada. 
    Se admiten los siguientes formatos de conjunto de datos:

    - Archivo comprimido en estas extensiones: ".zip", ".tar", ".gz" y ".bz2".
    - Carpeta que contiene imágenes. **Se recomienda encarecidamente comprimir esta carpeta primero y después usar el archivo comprimido como un conjunto de archivos**.

    > [!WARNING]
    > **No puede** usar el módulo **Importación de datos** para importar el conjunto de datos de imagen, porque el tipo de salida de dicho módulo es el directorio de DataFrames, que solo contiene la cadena de la ruta de acceso del archivo.
    

    > [!NOTE]
    > Si usa un conjunto de datos de imagen en el aprendizaje supervisado, se requiere la etiqueta.
    > En la tarea de clasificación de imágenes, la etiqueta se puede generar como "categoría" de imagen en la salida del módulo si este conjunto de datos de imagen está organizado en el formato ImageFolder de torchvision. De lo contrario, solo se guardan las imágenes sin etiqueta. A continuación se muestra un ejemplo de cómo puede organizar el conjunto de datos de imagen para obtener la etiqueta y usar la categoría de imagen como nombre de la subcarpeta. Consulte el artículo sobre los [conjuntos de datos de torchvision](https://pytorch.org/docs/stable/torchvision/datasets.html#imagefolder) para obtener más información.
    >
    > ```
    > root/dog/xxx.png
    > root/dog/xxy.png
    > root/dog/xxz.png
    >
    > root/cat/123.png
    > root/cat/nsdf3.png
    > root/cat/asd932_.png
    > ```

3.  Envíe la canalización.

## <a name="results"></a>Results

La salida del módulo **Conversión al directorio de imagen** está en formato de directorio de imagen y se puede conectar a otros módulos relacionados con imágenes cuyo formato de puerto de entrada también es el de directorio de imagen.

## <a name="technical-notes"></a>Notas técnicas 

###  <a name="expected-inputs"></a>Entradas esperadas  

| Nombre          | Tipo                  | Descripción   |
| ------------- | --------------------- | ------------- |
| Conjunto de datos de entrada | AnyDirectory, ZipFile | Conjunto de datos de entrada |

###  <a name="output"></a>Output  

| Nombre                   | Tipo           | Descripción            |
| ---------------------- | -------------- | ---------------------- |
| Directorio de imagen de salida | ImageDirectory | Directorio de imagen de salida |

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
