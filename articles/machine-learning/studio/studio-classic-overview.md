---
title: '¿Qué se puede hacer con Azure Machine Learning Studio (clásico): Azure?'
description: Machine Learning Studio (clásico) es una herramienta de arrastrar y colocar que permite crear rápidamente modelos desde una biblioteca de algoritmos y módulos listos para utilizarse.
services: machine-learning
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.assetid: e65c8fe1-7991-4a2a-86ef-fd80a7a06269
ms.service: machine-learning
ms.subservice: studio
ms.topic: overview
ms.date: 08/19/2020
ms.openlocfilehash: 1677bb5e7d94b92376bc42308b7bbcb0a8c24f97
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89039335"
---
# <a name="what-can-i-do-with-machine-learning-studio-classic"></a>¿Qué se puede hacer con Azure Machine Learning Studio (clásico)?

**SE APLICA A:**  ![sí](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)

[!INCLUDE [Designer notice](../../../includes/designer-notice.md)]

Machine Learning Studio (clásico) es una herramienta del tipo arrastrar y colocar que le permite crear, probar e implementar modelos de Machine Learning. Studio (clásico) publica modelos como servicios web que pueden consumir fácilmente en aplicaciones personalizadas o herramientas de inteligencia empresarial como Excel.


## <a name="studio-classic--interactive-workspace"></a>Área de trabajo interactiva de Studio (clásico)

Para desarrollar un modelo de análisis predictivo, normalmente se utilizan datos de una o varias fuentes, se transforman y analizan los datos a través de diversas funciones estadísticas y de manipulación de datos y se genera un conjunto de resultados. Desarrollar un modelo como este es un proceso iterativo: a medida que se modifican las diversas funciones y sus parámetros, sus resultados convergen hasta que esté satisfecho con un modelo entrenado y efectivo.

Machine Learning Studio (clásico) le proporciona un área de trabajo visual e interactiva para crear, probar e iterar fácilmente en un modelo de análisis predictivo. Los ***conjuntos de datos*** y los ***módulos*** de análisis se arrastran y colocan en un lienzo interactivo, y se conectan todos para formar un ***experimento*** que se ejecuta en Machine Learning Studio (clásico). Para iterar su diseño de modelo, se puede editar el experimento, guardar una copia si así se desea y ejecutarlo de nuevo. Cuando esté listo, puede convertir el ***experimento de entrenamiento*** en un ***experimento predictivo***, y luego publicarlo como ***servicio web*** para que otros usuarios puedan acceder al modelo.

No se requiere programación; conecte visualmente conjuntos de datos y módulos para construir el modelo de análisis predictivo.

![Diagrama de Machine Learning Studio (clásico): crear experimentos, leer datos de muchos orígenes, escribir datos puntuados, modelos de escritura.](./media/what-is-ml-studio/azure-ml-studio-diagram.jpg)

## <a name="download-the-ml-studio-classic-overview-diagram"></a>Descarga del diagrama de información general de Machine Learning Studio (clásico)
Descargue el diagrama **Microsoft Azure Machine Learning Studio (classic) Capabilities Overview** [Información general acerca de las funcionalidades de Machine Learning Studio (clásico)]. Para tenerlo a mano, puede imprimir el diagrama en tamaño tabloide (279 x 432 mm).

**Descargue el diagrama aquí: [Microsoft Azure Machine Learning Studio (classic) Capabilities Overview](https://download.microsoft.com/download/C/4/6/C4606116-522F-428A-BE04-B6D3213E9E52/ml_studio_overview_v1.1.pdf)** 
![Microsoft Azure Machine Learning Studio (classic) Capabilities Overview](./media/what-is-ml-studio/ml_studio_overview_v1.1.png) [Información general acerca de las funcionalidades de Machine Learning Studio (clásico)]


## <a name="components-of-a-studio-classic--experiment"></a>Componentes de un experimento de Studio (clásico)
Un experimento consta de conjuntos de datos que proporcionan datos a módulos analíticos, que se conectan en conjunto para construir un modelo de análisis predictivo. En concreto, un experimento válido tiene estas características:

* Tiene al menos un conjunto de datos y un módulo.
* Los conjuntos de datos pueden estar solo conectados a módulos.
* Los módulos pueden conectarse a conjuntos de datos o a otros módulos.
* Todos los puertos de entrada de los módulos deben tener alguna conexión al flujo de datos.
* Deben establecerse todos los parámetros necesarios para cada módulo.

Puede crear un experimento desde cero, o puede usar un experimento de ejemplo existente como plantilla. Para más información, consulte [Copia de experimentos de ejemplo para crear nuevos experimentos de aprendizaje automático](sample-experiments.md).

Para ver un ejemplo en el que se crea un experimento, consulte [Creación de un experimento en Machine Learning Studio (clásico)](create-experiment.md).

Para obtener un tutorial más completo de la creación de una solución de análisis predictivos, consulte [Desarrollo de una solución predictiva con Machine Learning Studio (clásico)](tutorial-part1-credit-risk.md).

### <a name="datasets"></a>Conjuntos de datos
Un conjunto de datos son los datos que se han cargado en Machine Learning Studio (clásico) para utilizarse en el proceso de modelado. Machine Learning Studio (clásico) incluye varios conjuntos de datos de ejemplo con los que puede experimentar, y puede cargar más a medida que los necesite. A continuación se muestran algunos ejemplos de los conjuntos de datos incluidos:

* **Datos sobre consumo de combustible por distancia recorrida para varios automóviles** : valores sobre consumo de combustible por distancia recorrida para automóviles identificados por el número de cilindros, los caballos de potencia, etc.
* **Datos sobre cáncer de mama** : datos de diagnóstico de cáncer de mama.
* **Datos de incendios forestales** : magnitud de los incendios forestales en el noreste de Portugal.

Cuando crea un experimento, puede elegir un conjunto de datos en la lista de conjuntos de datos disponibles a la izquierda del lienzo.

Para obtener una lista de los conjuntos de datos de ejemplo que se incluyen en Machine Learning Studio (clásico), vea [Uso de conjuntos de datos de ejemplo en Machine Learning Studio (clásico)](use-sample-datasets.md).

### <a name="modules"></a>Módulos
Un módulo es un algoritmo que puede aplicar sobre sus datos. Machine Learning Studio (clásico) cuenta con diversos módulos, que van desde las funciones de entrada de datos hasta procesos de entrenamiento, puntuación y validación. A continuación se muestran algunos ejemplos de los módulos incluidos:

* [Convertir a ARFF][convert-to-arff]: convierte un conjunto de datos serializados de .NET a Formato de archivo atributo-relación (ARFF).
* [Estadísticas elementales de procesos][elementary-statistics]: calcula estadísticas elementales como la media, la desviación estándar, etc.
* [Regresión lineal][linear-regression]: crea un modelo de regresión lineal basado en un descenso de gradiente en línea.
* [Puntuar modelo][score-model]: puntúa un modelo entrenado de clasificación o regresión.

Cuando crea un experimento, puede elegir un módulo en la lista de módulos disponibles a la izquierda del lienzo.

Un módulo puede tener un conjunto de parámetros que puede utilizar para configurar los algoritmos internos del módulo. Al seleccionar un módulo en el lienzo, los parámetros del módulo se muestran en el panel **Propiedades** a la derecha del lienzo. Puede modificar los parámetros en ese panel para ajustar su modelo.

Si necesita ayuda para desplazarse por la gran biblioteca de algoritmos de aprendizaje automático, consulte [Elección de algoritmos para Microsoft Machine Learning Studio (clásico)](algorithm-choice.md).

## <a name="deploying-a-predictive-analytics-web-service"></a>Implementación del servicio web de análisis predictivo
Una vez que el modelo de análisis predictivo esté listo, puede implementarlo como servicio web desde Machine Learning Studio (clásico). Para más información, consulte [Implementación de un servicio web de Azure Machine Learning](deploy-a-machine-learning-web-service.md).

## <a name="next-steps"></a>Pasos siguientes
Puede aprender lo básico del análisis predictivo y de Machine Learning mediante un [inicio rápido paso a paso](create-experiment.md) y el [uso de ejemplos](sample-experiments.md).

<!-- Module References -->
[convert-to-arff]: https://msdn.microsoft.com/library/azure/62d2cece-d832-4a7a-a0bd-f01f03af0960/
[elementary-statistics]: https://msdn.microsoft.com/library/azure/3086b8d4-c895-45ba-8aa9-34f0c944d4d3/
[linear-regression]: https://msdn.microsoft.com/library/azure/31960a6f-789b-4cf7-88d6-2e1152c0bd1a/
[score-model]: https://msdn.microsoft.com/library/azure/401b4f92-e724-4d5a-be81-d5b0ff9bdb33/
