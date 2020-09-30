---
title: Transformación de imagen init
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo Transformación de imagen init para inicializar la transformación de la imagen.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 05/26/2020
ms.openlocfilehash: aa81987f9214870e248ef9b625e6afcd1093fe5d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907885"
---
# <a name="init-image-transformation"></a>Transformación de imagen init

En este artículo se describe cómo usar el módulo **Transformación de imagen init** en el diseñador de Azure Machine Learning, para inicializar la transformación de la imagen con el fin de especificar cómo desea que se transforme la imagen.

## <a name="how-to-configure-init-image-transformation"></a>Configuración de la transformación de imagen init

1.  Agregue el módulo **Transformación de imagen init** a la canalización del diseñador. 

2.  En **Cambiar tamaño**, especifique si desea cambiar el tamaño de la imagen PIL de entrada al tamaño especificado. Si elige "Verdadero", puede especificar el tamaño deseado de la imagen de salida en **Tamaño**, de forma predeterminada 256. 

3.  En **Center crop** (Recorte en el centro), especifique si desea recortar la imagen PIL dada en el centro. Si elige "Verdadero", puede especificar el tamaño deseado de recorte de la imagen de salida en **Tamaño de recorte**, de forma predeterminada 224.  

4.  En **Relleno**, especifique si desea rellenar la imagen PIL dada en todos los lados con el valor de relleno 0. Si elige "Verdadero", puede especificar el relleno (número de píxeles que se agregarán) en cada borde en **Relleno**.

5.  En **Color jitter** (Inestabilidad de color), especifique si desea cambiar aleatoriamente el brillo, el contraste y la saturación de una imagen.

6.  En **Escala de grises**, especifique si desea convertir la imagen a escala de grises.

7.  En **Random resized crop** (Recorte de cambio de tamaño aleatorio), especifique si desea recortar la imagen de PIL dada a un tamaño aleatorio y una relación de aspecto. Se realiza un recorte de tamaño aleatorio (entre 0,08 y 1,0) del tamaño original y una relación de aspecto aleatoria (entre 3/4 y 4/3) de la relación de aspecto original. Por último, se cambia el tamaño de este recorte a un tamaño determinado.
    Se usa normalmente en el entrenamiento de las redes de origen. Si elige "Verdadero", puede especificar el tamaño de salida esperado de cada borde en **Random size** (Tamaño aleatorio), de forma predeterminada 256.

8.  En **Random crop** (Recorte aleatorio), especifique si quiere recortar la imagen PIL determinada en una ubicación aleatoria. Si elige "Verdadero", puede especificar el tamaño de salida deseado del recorte en **Random crop size** (Tamaño de recorte aleatorio), de forma predeterminada 224.

9.  En **Random horizontal flip** (Volteo horizontal aleatorio), especifique si desea voltear horizontalmente la imagen PIL dada de forma aleatoria con la probabilidad de 0,5.

10.  En **Random vertical flip** (Volteo vertical aleatorio), especifique si desea voltear verticalmente la imagen PIL dada de forma aleatoria con la probabilidad de 0,5.

11.  En **Random rotation** (Rotación aleatoria), especifique si desea girar la imagen por ángulo. Si elige "Verdadero", se puede especificar en intervalo de grados estableciendo **Random rotation degrees** (Grados de giro aleatorio), lo que significa (-grados, +grados), de forma predeterminada 0.

12.  En **Random affine** (Afín aleatoria), especifique si desea realizar una transformación afín aleatoria de la imagen conservando el centro invariable. Si elige "Verdadero", se puede especificar en intervalo de grados que se puede seleccionar en **Random affine degrees** (Grados de afinidad aleatoria), lo que significa (-grados, +grados), de forma predeterminada 0.

13.  En **Random grayscale** (Escala de grises aleatoria), especifique si quiere convertir la imagen de forma aleatoria a escala de grises con una probabilidad de 0,1.

14.  En **Random perspective** (Perspectiva aleatoria), especifique si desea realizar la transformación de la perspectiva de la imagen PIL dada aleatoriamente con una probabilidad de 0,5.


16.  Conéctese al módulo [Aplicación de transformación de imagen](apply-image-transformation.md), para aplicar la transformación especificada anteriormente al conjunto de datos de la imagen de entrada.

17. Envíe la canalización.

## <a name="results"></a>Results

Una vez completada la transformación, puede encontrar las imágenes transformadas en la salida del módulo [Aplicación de transformación de imagen](apply-image-transformation.md).


## <a name="technical-notes"></a>Notas técnicas  

Consulte [https://pytorch.org/docs/stable/torchvision/transforms.html](https://pytorch.org/docs/stable/torchvision/transforms.html) para más información sobre la transformación de imágenes.

###  <a name="module-parameters"></a>Parámetros del módulo  

| Nombre                    | Intervalo   | Tipo    | Valor predeterminado | Descripción                              |
| ----------------------- | ------- | ------- | ------- | ---------------------------------------- |
| Cambiar de tamaño                  | Any     | Boolean | True    | Cambiar el tamaño de la imagen PIL de entrada al tamaño especificado |
| Size                    | >=1     | Entero | 256     | Especificar el tamaño de salida deseado          |
| Recorte en el centro             | Any     | Boolean | True    | Recortar la imagen PIL dada en el centro  |
| Tamaño de recorte               | >=1     | Entero | 224     | Especificar el tamaño de salida deseado del recorte |
| Pad                     | Any     | Boolean | False   | Rellenar la imagen PIL determinado en todos los lados con el valor de "relleno" dado |
| Relleno                 | >=0     | Entero | 0       | Rellenar en cada borde                   |
| Inestabilidad de color            | Any     | Boolean | False   | Cambiar aleatoriamente el brillo, el contraste y la saturación de una imagen |
| Escala de grises               | Any     | Boolean | False   | Convertir imagen en escala de grises               |
| Recorte con cambio de tamaño aleatorio     | Any     | Boolean | False   | Recortar la imagen PIL dada al tamaño aleatorio y la relación de aspecto |
| Tamaño aleatorio             | >=1     | Entero | 256     | Tamaño de salida esperado de cada borde        |
| Recorte aleatorio             | Any     | Boolean | False   | Recortar la imagen PIL dada en una ubicación aleatoria |
| Tamaño de recorte aleatorio        | >=1     | Entero | 224     | Tamaño de salida deseado del recorte          |
| Volteo horizontal aleatorio  | Any     | Boolean | True    | Voltear horizontalmente la imagen PIL dada de forma aleatoria con una probabilidad determinada |
| Volteo vertical aleatorio    | Any     | Boolean | False   | Voltear verticalmente la imagen PIL dada de forma aleatoria con una probabilidad determinada |
| Rotación aleatoria         | Any     | Boolean | False   | Girar la imagen por ángulo                |
| Grados de rotación aleatoria | [0,180] | Entero | 0       | Intervalo de grados entre los que se puede seleccionar          |
| Afín aleatoria           | Any     | Boolean | False   | Transformación afín aleatoria de la imagen conservando el centro invariable |
| Grados de afinidad aleatoria   | [0,180] | Entero | 0       | Intervalo de grados entre los que se puede seleccionar          |
| Escala de grises aleatoria        | Any     | Boolean | False   | Convertir la imagen de forma aleatoria a escala de grises con una probabilidad de 0,1 |
| Perspectiva aleatoria      | Any     | Boolean | False   | Realizar la transformación de la perspectiva de la imagen PIL dada aleatoriamente con una probabilidad de 0,5 |
| Borrado aleatorio          | Any     | Boolean | False   | Seleccionar aleatoriamente una región de rectángulo en una imagen y borrar sus píxeles con una probabilidad de 0,5 |

###  <a name="output"></a>Output  

| Nombre                        | Tipo                    | Descripción                              |
| --------------------------- | ----------------------- | ---------------------------------------- |
| Transformación de imagen de salida | TransformationDirectory | Transformación de la imagen de salida que se puede conectar al módulo **Aplicación de transformación de imagen**. |

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 