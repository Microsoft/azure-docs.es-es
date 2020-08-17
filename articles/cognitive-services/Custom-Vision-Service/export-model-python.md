---
title: 'Tutorial: Ejecución del modelo de TensorFlow en Python con Custom Vision Service'
titleSuffix: Azure Cognitive Services
description: Ejecución de un modelo de TensorFlow en Python Este artículo solo se aplica a los modelos exportados desde proyectos de clasificación de imágenes en Custom Vision Service.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: tutorial
ms.date: 04/14/2020
ms.author: pafarley
ms.custom: tracking-python
ms.openlocfilehash: 8e2df90c2e0d3c0d6fd24884a46156de55ac55ec
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87079217"
---
# <a name="tutorial-run-tensorflow-model-in-python"></a>Tutorial: Ejecución del modelo de TensorFlow en Python

Tras la [exportación del modelo de TensorFlow](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/export-your-model) desde Custom Vision Service, esta guía de inicio rápido le mostrará cómo usar este modelo localmente para clasificar imágenes.

> [!NOTE]
> Este tutorial se aplica solo a los modelos exportados desde proyectos de clasificación de imágenes.

## <a name="prerequisites"></a>Prerequisites

Para usar el tutorial, debe realizar los siguientes pasos:

- Instale Python 2.7 o posterior o Python 3.5 o posterior.
- Instale PIP.

A continuación, deberá instalar los siguientes paquetes:

```
pip install tensorflow
pip install pillow
pip install numpy
pip install opencv-python
```

## <a name="load-your-model-and-tags"></a>Carga del modelo y de las etiquetas

El archivo zip descargado contiene un archivo model.pb y un archivo labels.txt. Estos archivos representan el modelo entrenado y las etiquetas de clasificación. El primer paso consiste en cargar el modelo en el proyecto.

```Python
import tensorflow as tf
import os

graph_def = tf.compat.v1.GraphDef()
labels = []

# These are set to the default names from exported models, update as needed.
filename = "model.pb"
labels_filename = "labels.txt"

# Import the TF graph
with tf.io.gfile.GFile(filename, 'rb') as f:
    graph_def.ParseFromString(f.read())
    tf.import_graph_def(graph_def, name='')

# Create a list of labels.
with open(labels_filename, 'rt') as lf:
    for l in lf:
        labels.append(l.strip())
```

## <a name="prepare-an-image-for-prediction"></a>Preparación de una imagen para la predicción

Hay algunos pasos que debe seguir para preparar la imagen para la predicción. Estos pasos imitan la manipulación de imágenes realizada durante el entrenamiento:

### <a name="open-the-file-and-create-an-image-in-the-bgr-color-space"></a>Apertura del archivo y creación de una imagen en el espacio de colores BGR

```Python
from PIL import Image
import numpy as np
import cv2

# Load from a file
imageFile = "<path to your image file>"
image = Image.open(imageFile)

# Update orientation based on EXIF tags, if the file has orientation info.
image = update_orientation(image)

# Convert to OpenCV format
image = convert_to_opencv(image)
```

### <a name="handle-images-with-a-dimension-1600"></a>Tratamiento de las imágenes con una dimensión mayor que 1600

```Python
# If the image has either w or h greater than 1600 we resize it down respecting
# aspect ratio such that the largest dimension is 1600
image = resize_down_to_1600_max_dim(image)
```

### <a name="crop-the-largest-center-square"></a>Recorte del cuadrado central más grande

```Python
# We next get the largest center square
h, w = image.shape[:2]
min_dim = min(w,h)
max_square_image = crop_center(image, min_dim, min_dim)
```

### <a name="resize-down-to-256x256"></a>Cambio de tamaño hasta 256 x 256

```Python
# Resize that square down to 256x256
augmented_image = resize_to_256_square(max_square_image)
```

### <a name="crop-the-center-for-the-specific-input-size-for-the-model"></a>Recorte del centro para el tamaño de entrada específico del modelo

```Python
# Get the input size of the model
with tf.compat.v1.Session() as sess:
    input_tensor_shape = sess.graph.get_tensor_by_name('Placeholder:0').shape.as_list()
network_input_size = input_tensor_shape[1]

# Crop the center for the specified network_input_Size
augmented_image = crop_center(augmented_image, network_input_size, network_input_size)

```

En los pasos anteriores se usan las siguientes funciones auxiliares:

```Python
def convert_to_opencv(image):
    # RGB -> BGR conversion is performed as well.
    image = image.convert('RGB')
    r,g,b = np.array(image).T
    opencv_image = np.array([b,g,r]).transpose()
    return opencv_image

def crop_center(img,cropx,cropy):
    h, w = img.shape[:2]
    startx = w//2-(cropx//2)
    starty = h//2-(cropy//2)
    return img[starty:starty+cropy, startx:startx+cropx]

def resize_down_to_1600_max_dim(image):
    h, w = image.shape[:2]
    if (h < 1600 and w < 1600):
        return image

    new_size = (1600 * w // h, 1600) if (h > w) else (1600, 1600 * h // w)
    return cv2.resize(image, new_size, interpolation = cv2.INTER_LINEAR)

def resize_to_256_square(image):
    h, w = image.shape[:2]
    return cv2.resize(image, (256, 256), interpolation = cv2.INTER_LINEAR)

def update_orientation(image):
    exif_orientation_tag = 0x0112
    if hasattr(image, '_getexif'):
        exif = image._getexif()
        if (exif != None and exif_orientation_tag in exif):
            orientation = exif.get(exif_orientation_tag, 1)
            # orientation is 1 based, shift to zero based and flip/transpose based on 0-based values
            orientation -= 1
            if orientation >= 4:
                image = image.transpose(Image.TRANSPOSE)
            if orientation == 2 or orientation == 3 or orientation == 6 or orientation == 7:
                image = image.transpose(Image.FLIP_TOP_BOTTOM)
            if orientation == 1 or orientation == 2 or orientation == 5 or orientation == 6:
                image = image.transpose(Image.FLIP_LEFT_RIGHT)
    return image
```

## <a name="predict-an-image"></a>Predicción de una imagen

Una vez preparada la imagen como un tensor, podemos enviarla a través del modelo para una predicción:

```Python

# These names are part of the model and cannot be changed.
output_layer = 'loss:0'
input_node = 'Placeholder:0'

with tf.compat.v1.Session() as sess:
    try:
        prob_tensor = sess.graph.get_tensor_by_name(output_layer)
        predictions = sess.run(prob_tensor, {input_node: [augmented_image] })
    except KeyError:
        print ("Couldn't find classification output layer: " + output_layer + ".")
        print ("Verify this a model exported from an Object Detection project.")
        exit(-1)
```

## <a name="view-the-results"></a>View the results

Los resultados de la ejecución del tensor de la imagen a través del modelo tendrán que volver a asignarse a las etiquetas.

```Python
    # Print the highest probability label
    highest_probability_index = np.argmax(predictions)
    print('Classified as: ' + labels[highest_probability_index])
    print()

    # Or you can print out all of the results mapping labels to probabilities.
    label_index = 0
    for p in predictions:
        truncated_probablity = np.float64(np.round(p,8))
        print (labels[label_index], truncated_probablity)
        label_index += 1
```

## <a name="next-steps"></a>Pasos siguientes

A continuación, obtenga información sobre cómo encapsular el modelo en una aplicación móvil:
* [Use your exported Tensorflow model in an Android application](https://github.com/Azure-Samples/cognitive-services-android-customvision-sample) (Uso del modelo Tensorflow exportado en una aplicación de Android)
* [Use your exported CoreML model in a Swift iOS application](https://go.microsoft.com/fwlink/?linkid=857726) (Uso del modelo CoreML exportado en una aplicación de iOS con Swift)
* [Use your exported CoreML model in an iOS application with Xamarin](https://github.com/xamarin/ios-samples/tree/master/ios11/CoreMLAzureModel) (Uso del modelo CoreML exportado en una aplicación de iOS con Xamarin)
