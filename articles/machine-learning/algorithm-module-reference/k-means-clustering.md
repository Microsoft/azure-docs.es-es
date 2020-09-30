---
title: 'Agrupación en clústeres K-Means: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Aprenda a usar el módulo Agrupación en clústeres K-Means en Azure Machine Learning para entrenar modelos de agrupación en clústeres.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 08/04/2020
ms.openlocfilehash: 97cadfb8f5004cfd2701335172d4416c64f05259
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907872"
---
# <a name="module-k-means-clustering"></a>Módulo: Agrupación en clústeres K-Means

En este artículo se describe cómo usar el módulo *Agrupación en clústeres K-Means* en el diseñador de Azure Machine Learning Studio para crear un modelo de agrupación en clústeres K-means no entrenado. 
 
K-means es uno de los algoritmos de aprendizaje *supervisados* más sencillos y el más conocido. Puede usar el algoritmo para diversas tareas de aprendizaje automático, como son: 

* [Detección de datos anormales](https://msdn.microsoft.com/magazine/jj891054.aspx).
* Agrupación en clústeres de documentos de texto.
* Análisis de conjuntos de datos antes de utilizar otros métodos de clasificación o regresión. 

Para crear un modelo de agrupación en clústeres:

* Agregue este módulo a la canalización.
* Conéctese a un conjunto de datos.
* Configure los parámetros, como el número de clústeres que prevé, la métrica de distancia que se usa para crear los clústeres, etc. 
  
Después de configurar los hiperparámetros de módulo, conecte el modelo no entrenado al módulo [Train Clustering Model](train-clustering-model.md) (Entrenamiento del modelo de agrupación en clústeres). Dado que el algoritmo K-means es un método de aprendizaje no supervisado, la columna de etiqueta es opcional. 

+ Si los datos incluyen una etiqueta, puede usar los valores de etiqueta para guiar la selección de los clústeres y optimizar el modelo. 

+ Si los datos no tienen ninguna etiqueta, el algoritmo crea clústeres que representan las categorías posibles, basadas únicamente en los datos.  

##  <a name="understand-k-means-clustering"></a>Agrupación en clústeres K-Means
 
En general, la agrupación en clústeres utiliza técnicas iterativas para agrupar los casos de un conjunto de datos en clústeres que poseen características similares. Estas agrupaciones son útiles para explorar los datos, identificar anomalías en ellos y, finalmente, para realizar predicciones. Los modelos de agrupación en clústeres también pueden ayudarle a identificar las relaciones en un conjunto de datos que podrían no deducirse lógicamente mediante una simple observación o examen. Por estos motivos, la agrupación en clústeres se suele usar en las primeras fases de las tareas de aprendizaje automático, a fin de explorar los datos y detectar correlaciones inesperadas.  
  
 Cuando se configura un modelo de agrupación en clústeres mediante el método K-means, debe especificar un número de destino *k* que indica el número de *centroides* que desee en el modelo. El centroide es un punto representativo de cada clúster. El algoritmo K-means asigna cada punto de datos entrante a uno de los clústeres minimizando la suma en el clúster de cuadrados. 
 
Cuando procesa los datos de entrenamiento, el algoritmo K-means comienza con un conjunto inicial de centroides elegidos al azar. Los centroides sirven como punto de partida para los clústeres y se aplica el algoritmo de Lloyd para refinar sus ubicaciones de manera iterativa. El algoritmo K-means deja de generar y refinar los clústeres cuando cumplen una o varias de estas condiciones:  
  
-   Los centroides se estabilizan, lo que significa que las asignaciones de clúster para puntos individuales ya no podrán cambiar y el algoritmo ha convergido en una solución.  
  
-   El algoritmo se completó ejecutando el número de iteraciones especificado.  
  
 Tras completar la fase de entrenamiento, se usa el módulo [Asignación de datos a clústeres](assign-data-to-clusters.md) para asignar nuevos casos a uno de los clústeres que encontró con el algoritmo K-means. Para realizar la asignación de clúster, calcule la distancia entre el nuevo caso y el centroide de cada clúster. Cada nuevo caso se asigna al clúster con el centroide más cercano.  

## <a name="configure-the-k-means-clustering-module"></a>Configuración del módulo de agrupación en clústeres K-Means
  
1.  Agregue el módulo **Agrupación en clústeres K-Means** a la canalización.  
  
2.  Para especificar cómo quiere que se entrene el modelo, seleccione la opción **Create trainer mode** (Crear modo entrenador).  
  
    -   **Single Parameter** (Parámetro único): si conoce los parámetros exactos que desea usar en el modelo de agrupación en clústeres, puede proporcionar un conjunto específico de valores como argumentos.  
  
3.  En **Number of centroids** (Número de centroides), escriba el número de clústeres con el que desea que comience el algoritmo.  
  
     No se garantiza que el modelo genere exactamente este número de clústeres. El algoritmo comienza con este número de puntos de datos e itera para buscar la configuración óptima. Puede hacer referencia al [código fuente de sklearn](https://github.com/scikit-learn/scikit-learn/blob/fd237278e/sklearn/cluster/_kmeans.py#L1069).
  
4.  La **inicialización** de las propiedades se usa para especificar el algoritmo que se emplea para definir la configuración inicial del clúster.  
  
    -   **First N** (Primeros N): Se elige un número de puntos de datos inicial en el conjunto de datos y se usa como la media inicial. 
    
         Este método también se conoce como *método Forgy*.  
  
    -   **Random** (Aleatorio): el algoritmo coloca aleatoriamente un punto de datos en un clúster y, a continuación, calcula la media inicial para que sea el centroide de los puntos asignados de forma aleatoria del clúster. 

         Este método también se denomina de *partición aleatoria*.  
  
    -   **K-Means++** : este es el método predeterminado para inicializar los clústeres.  
  
         El algoritmo **K-means ++** fue propuesto en 2007 por David Arthur y Sergei Vassilvitskii para evitar una agrupación en clústeres deficiente del algoritmo K-means estándar. **K-means ++** mejora el método K-means estándar con un método diferente para elegir los centros iniciales del clúster.  
  
    
5.  Como **Random number seed** (Valor de inicialización aleatorio), puede escribir el valor que se usará como el inicial para la inicialización de clúster. Este valor puede tener un efecto significativo en la selección del clúster.  
  
6.  En **Metric** (Métrica), elija la función que se usará para medir la distancia entre los vectores de clúster o entre los nuevos puntos de datos y el centroide elegido al azar. Azure Machine Learning admite las siguientes métricas de distancia de clúster:  
  
    -   **Euclidian** (Euclidiana): la distancia Euclidiana se utiliza normalmente como una medida de dispersión de clúster para la agrupación en clústeres K-means. Se prefiere el uso de esta métrica porque minimiza la distancia media entre los puntos y los centroides.
  
7.  En **Iterations** (Iteraciones), escriba el número de veces que el algoritmo debe iterar sobre los datos de entrenamiento antes de finalizar la selección de centroides.  
  
     Puede ajustar este parámetro para equilibrar la precisión en el tiempo de entrenamiento.  
  
8.  En **Assign label mode** (Modo de asignación de etiqueta), elija una opción que especifique cómo se debe tratar una columna de etiqueta, si está presente en el conjunto de datos.  
  
     Dado que la agrupación en clústeres K-means es un método de aprendizaje de automático no supervisado, las etiquetas son opcionales. Sin embargo, si el conjunto de datos ya tiene una columna de etiqueta, puede usar esos valores para guiar la selección de los clústeres o bien puede especificar que los valores se omitan.  
  
    -   **Ignore label column** (Ignorar la columna de etiqueta): los valores de la columna de etiqueta se omiten y no se usan en la creación del modelo.
  
    -   **Fill missing values** (Completar los valores que faltan: los valores de la columna de etiqueta se usan como características para ayudar a crear los clústeres. Si en alguna de las filas falta una etiqueta, el valor se deduce con otras características.  
  
    -   **Overwrite from closest to center** (Sobrescribir desde la más cercana al centro): los valores de la columna de etiqueta son reemplazados por los valores de la etiqueta predicha, utilizando la etiqueta del punto más cercana al centroide actual.  

8.  Seleccione la opción **Normalize features** (Normalizar características) si desea normalizar las características antes del entrenamiento.
  
     Si aplica la normalización, antes del entrenamiento, los puntos de datos son normalizados en `[0,1]` con MinMaxNormalizer.

10. Entrene el modelo.  
  
    -   Si establece **Create trainer mode** (Crear modo de entrenador) en **Single Parameter** (Parámetro único), agregue un conjunto de datos etiquetado y entrene el modelo con el módulo [Train Clustering Model](train-clustering-model.md) (Entrenamiento del modelo de agrupación en clústeres).  
  
## <a name="results"></a>Results

Cuando haya terminado de configurar y entrenar el modelo, puede usarlo para generar puntuaciones. Sin embargo, hay varias formas de entrenar el modelo y varias maneras de ver y usar los resultados: 

### <a name="capture-a-snapshot-of-the-model-in-your-workspace"></a>Captura de una instantánea del modelo en el área de trabajo

Si usó el módulo [Train Clustering Model clústeres](train-clustering-model.md) (Entrenamiento del modelo de agrupación en clústeres):

1. Seleccione el módulo **Train Clustering Model** (Entrenar modelo de agrupación en clústeres) y abra el panel derecho.

2. Seleccione la pestaña **Outputs** (Salidas). Seleccione el icono **Register dataset** (Registrar conjunto de datos) para guardar una copia del modelo entrenado.

El modelo guardado representa los datos de entrenamiento en el momento de guardar el modelo. Si más tarde actualiza los datos de entrenamiento utilizados en la canalización, no actualiza el modelo guardado. 

### <a name="see-the-clustering-result-dataset"></a>Ver el conjunto de datos de resultado de la agrupación en clústeres 

Si usó el módulo [Train Clustering Model clústeres](train-clustering-model.md) (Entrenamiento del modelo de agrupación en clústeres):

1. Haga clic con el botón secundario en el módulo **Train Clustering Model clústeres** (Entrenamiento del modelo de agrupación en clústeres).

2. Seleccione **Visualize** (Visualizar).

### <a name="tips-for-generating-the-best-clustering-model"></a>Sugerencias para generar el mejor modelo de agrupación en clústeres  

Se sabe que el proceso de *inicialización* que se usa durante la agrupación en clústeres puede afectar significativamente al modelo. La inicialización implica la ubicación inicial de puntos en los centroides posibles.
 
Por ejemplo, si el conjunto de datos contiene muchos valores atípicos y se elige uno para propagar los clústeres, ningún otro punto de datos encajaría bien con ese clúster y este puede ser un singleton. Es decir, podría tener un solo punto.  
  
Puede evitar este problema de dos maneras:  
  
-   Cambie el número de centroides y pruebe con varios valores de inicialización.  
  
-   Cree varios modelos, variando la métrica o la iteración más.  
  
En general, con los modelos de clústeres, es posible que una determinada configuración de como resultado un conjunto de clústeres optimizado localmente. En otras palabras, el conjunto de clústeres que devuelve el modelo se adapta solo a los puntos de datos actuales y no es generalizable a otros datos. Si usa una configuración inicial diferente, el método K-means podría encontrar una configuración diferente y superior. 

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
