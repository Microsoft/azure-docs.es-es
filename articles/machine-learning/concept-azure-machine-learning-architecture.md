---
title: Arquitectura y conceptos clave
titleSuffix: Azure Machine Learning
description: Obtenga información sobre la arquitectura, los términos, los conceptos y los flujos de trabajo que conforman Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: larryfr
author: Blackmist
ms.date: 05/13/2020
ms.custom: seoapril2019, seodec18
ms.openlocfilehash: 749a2366438bd1abfef4ca0cf2a195f23529d6a5
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86536307"
---
# <a name="how-azure-machine-learning-works-architecture-and-concepts"></a>Funcionamiento de Azure Machine Learning: Arquitectura y conceptos

Obtenga información sobre la arquitectura, los conceptos y el flujo de trabajo de Azure Machine Learning. En el siguiente diagrama se muestran los componentes principales del servicio y se ilustra el flujo de trabajo general al usar el servicio:

![Arquitectura y flujo de trabajo de Azure Machine Learning](./media/concept-azure-machine-learning-architecture/workflow.png)

## <a name="workflow"></a>Flujo de trabajo

Normalmente, el flujo de trabajo del modelo de Machine Learning sigue estos pasos:

1. **Entrenar**
    + Desarrolle scripts de entrenamiento de aprendizaje automático en **Python**, **R** o con el diseñador visual.
    + Se crea y configura un **destino de proceso**.
    + **Envíe los scripts** a un destino de proceso configurado para ejecutarse en ese entorno. Durante el entrenamiento, los scripts pueden leer o escribir en **almacenes de datos**. Los registros y las salidas generados durante el entrenamiento se guardan como **ejecuciones** en el **área de trabajo** y se agrupan en **experimentos**.

1. **Paquete**: después de encontrar una ejecución satisfactoria, se registra el modelo guardado en el **registro de modelos**.

1. **Validar** - **Consulte el experimento** para las métricas registradas en ejecuciones actuales y anteriores. Si las métricas no ofrecen el resultado deseado, se vuelve en bucle al paso 1 y se repiten los scripts.

1. **Implementar**: desarrolle un script de puntuación que usa el modelo e **implemente el modelo** como un **servicio web** en Azure, o en un **dispositivo IoT Edge**.

1. **Supervisión**: supervise el **desfase de datos** entre el conjunto de datos de entrenamiento y los datos de inferencia de un modelo implementado. Cuando sea necesario, vuelva al paso 1 para volver a entrenar el modelo con nuevos datos de entrenamiento.

## <a name="tools-for-azure-machine-learning"></a>Herramientas para Azure Machine Learning

Utilice estas herramientas para Azure Machine Learning:

> [!IMPORTANT]
> Las herramientas marcadas (versión preliminar) a continuación se encuentran actualmente en versión preliminar pública.
> Se ofrece la versión preliminar sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

+  Interactúe con el servicio en cualquier entorno de Python con el [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py).
+ Interactúe con el servicio en cualquier entorno de R con el [SDK de Azure Machine Learning para R](https://azure.github.io/azureml-sdk-for-r/reference/index.html) (versión preliminar).
+ Automatice las actividades de aprendizaje automático con la [CLI de Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/reference-azure-machine-learning-cli).
+ Use el [diseñador de Azure Machine Learning (versión preliminar)](concept-designer.md) para realizar los pasos del flujo de trabajo sin escribir código. (Para usar el diseñador se necesita un [área de trabajo Enterprise](concept-workspace.md#upgrade)).
+ El [Acelerador de soluciones Many Models](https://aka.ms/many-models) (versión preliminar) se basa en Azure Machine Learning y permite entrenar, usar y administrar cientos o incluso miles de modelos de Machine Learning.

> [!NOTE]
> Aunque en este artículo se definen los términos y conceptos que usa Azure Machine Learning, no se definen los términos y conceptos de la plataforma Azure. Para obtener más información sobre la terminología de la plataforma Azure, consulte el [glosario de Microsoft Azure](https://docs.microsoft.com/azure/azure-glossary-cloud-terminology).

## <a name="glossary"></a>Glosario

* [Actividad](#activities)
* [Área de trabajo](#workspaces)
    * [Experimentos](#experiments)
        * [Ejecutar](#runs) 
            * [Configuración de ejecución](#run-configurations)
            * [Instantánea](#snapshots)
            * [Seguimiento de GIT](#github-tracking-and-integration)
            * [Logging](#logging)
    * [Canalizaciones de Machine Learning](#ml-pipelines)
    * [Modelos](#models)
        * [Entornos](#environments)
        * [Script de entrenamiento](#training-scripts)
        * [Estimadores](#estimators)
    * [Extremos](#endpoints)
        * [Servicio web](#web-service-endpoint)
        * [Módulos de IoT](#iot-module-endpoints)
    * [Conjunto de datos y almacenes de datos](#datasets-and-datastores)
    * [Destinos de proceso](#compute-targets)

### <a name="activities"></a>Actividades

Una actividad representa una operación de larga ejecución. Las operaciones siguientes son ejemplos de actividades:

* Creación o eliminación de un destino de proceso
* Ejecución de un script en un destino de proceso

Las actividades pueden proporcionar notificaciones a través del SDK o la interfaz de usuario web para que pueda supervisar fácilmente el progreso de estas operaciones.

### <a name="workspaces"></a>Áreas de trabajo

[El área de trabajo](concept-workspace.md) es el recurso de nivel superior de Azure Machine Learning. Proporciona un lugar centralizado para trabajar con todos los artefactos que cree al usar Azure Machine Learning. Puede compartir un área de trabajo con otros usuarios. Para una descripción detallada de las áreas de trabajo, consulte [¿Qué es un área de trabajo de Azure Machine Learning?](concept-workspace.md)

### <a name="experiments"></a>Experimentos

Un experimento es una agrupación de varias ejecuciones de un script determinado. Siempre pertenece a un área de trabajo. Cuando envíe una ejecución, proporcione un nombre de experimento. La información de la ejecución se almacena en ese experimento. Si envía una ejecución y especifica un nombre de experimento que no existe, se crea automáticamente un experimento nuevo con ese nombre.

Para obtener un ejemplo del uso de un experimento, consulte [Tutorial: Entrenamiento del primer modelo](tutorial-1st-experiment-sdk-train.md).

### <a name="runs"></a>Ejecuciones

Una ejecución hace referencia a una única ejecución de un script de entrenamiento. Un experimento normalmente contendrá varias ejecuciones.

Azure Machine Learning registra todas las ejecuciones y almacena la siguiente información en el experimento:

* Metadatos sobre la ejecución (marca de tiempo, duración, etc.).
* Métricas que el script registra.
* Archivos de salida que recopila automáticamente el experimento o que carga explícitamente usted mismo.
* Instantánea del directorio que contiene los scripts antes de la ejecución.

Una ejecución se produce cuando se envía un script para entrenar un modelo. Una ejecución puede tener cualquier número de ejecuciones secundarias. Por ejemplo, la ejecución de nivel superior puede tener dos ejecuciones secundarias, cada una de las cuales puede tener sus propias ejecuciones secundarias.

### <a name="run-configurations"></a>Configuraciones de ejecución

Una configuración de ejecución es un conjunto de instrucciones que define cómo se debe ejecutar un script en un destino de proceso determinado. Esta configuración incluye un amplio conjunto de definiciones de comportamiento como, por ejemplo, si quiere usar un entorno de Python existente o un entorno de Conda creado a partir de la especificación.

Una configuración de ejecución puede conservarse en un archivo del directorio que contiene el script de entrenamiento o puede crearse como un objeto en la memoria y usarse para enviar una ejecución.

Para ver configuraciones de ejecución de ejemplo, consulte [Selección y uso de un destino de proceso para entrenar el modelo](how-to-set-up-training-targets.md).

### <a name="snapshots"></a>Instantáneas

Al enviar una ejecución, Azure Machine Learning comprime el directorio que contiene el script como un archivo zip y lo envía al destino de proceso. A continuación, el archivo .zip se extrae y el script se ejecuta. Azure Machine Learning también almacena el archivo .zip como una instantánea como parte del registro de ejecución. Cualquier persona con acceso al área de trabajo puede buscar un registro de ejecución y descargar la instantánea.

> [!NOTE]
> [!INCLUDE [amlinclude-info](../../includes/machine-learning-amlignore-gitignore.md)]

### <a name="github-tracking-and-integration"></a>Integración y seguimiento de GitHub

Cuando se inicia una ejecución de entrenamiento en la que el directorio de origen es un repositorio de GIT local, se almacena información sobre el repositorio en el historial de ejecución. Esto funciona con ejecuciones enviadas mediante un estimador, una canalización de aprendizaje automático o una ejecución del script. También funciona para las ejecuciones enviadas desde el SDK o la CLI de Machine Learning.

Para más información, consulte [Integración de Git con Azure Machine Learning](concept-train-model-git-integration.md).

### <a name="logging"></a>Registro

Al desarrollar la solución, use el SDK de Python de Azure Machine Learning en el script de Python para registrar las métricas arbitrarias. Después de la ejecución, consulte las métricas para determinar si la ejecución ha generado el modelo que quiere implementar.

### <a name="ml-pipelines"></a>Canalizaciones de Machine Learning

Las canalizaciones de aprendizaje automático se usan para crear y administrar flujos de trabajo que unen las fases de aprendizaje automático. Por ejemplo, una canalización podría incluir las fases de preparación de los datos, entrenamiento del modelo, implementación del modelo e inferencia y puntuación. Cada fase puede estar formada por varios pasos, cada uno de los cuales puede ejecutarse en modo desatendido en varios destinos de proceso. 

Los pasos de canalización se pueden reutilizar y se pueden ejecutar sin volver a ejecutar los pasos anteriores si la salida de esos pasos no ha cambiado. Por ejemplo, puede volver a entrenar un modelo sin volver a ejecutar los costosos pasos de preparación de datos si los datos no han cambiado. Las canalizaciones también permiten a los científicos de datos colaborar mientras trabajan en áreas independientes de un flujo de trabajo de Machine Learning.

Para obtener más información sobre las canalizaciones de aprendizaje automático con este servicio, consulte el artículo [Canalizaciones y Azure Machine Learning](concept-ml-pipelines.md).

### <a name="models"></a>Modelos

En su forma más simple, un modelo es un fragmento de código que toma una entrada y genera una salida. El hecho de crear un modelo de Machine Learning implica seleccionar un algoritmo, proporcionarle datos y ajustar los hiperparámetros. El entrenamiento es un proceso iterativo que genera un modelo entrenado, que encapsula lo que el modelo ha aprendido durante el proceso de aprendizaje.

Un modelo se genera mediante una ejecución en Azure Machine Learning. También puede usar un modelo entrenado fuera de Azure Machine Learning. Puede registrar el modelo en un área de trabajo de Azure Machine Learning.

Azure Machine Learning es independiente del marco de trabajo. Al crear un modelo, puede usar cualquier marco de aprendizaje automático popular, como Scikit-learn, XGBoost, PyTorch, TensorFlow y Chainer.

Para ver un ejemplo de cómo entrenar un modelo con Scikit-learn y un estimador, consulte [Tutorial: Entrenamiento de un modelo de clasificación de imágenes con Azure Machine Learning](tutorial-train-models-with-aml.md).

El **registro de modelos** realiza un seguimiento de todos los modelos del área de trabajo de Azure Machine Learning.

Los modelos se identifican por el nombre y la versión. Cada vez que registra un modelo con el mismo nombre que uno ya existente, el registro interpreta que se trata de una nueva versión. La versión se incrementa y el nuevo modelo se registra con el nombre en cuestión.

Una vez registrado el modelo, puede proporcionar etiquetas de metadatos adicionales y, después, usar esas etiquetas al buscar modelos.

> [!TIP]
> Un modelo registrado es un contenedor lógico para uno o varios archivos que componen el modelo. Por ejemplo, si tiene un modelo que se almacena en varios archivos, puede registrarlos como un único modelo en el área de trabajo de Azure Machine Learning. Después del registro, puede descargar o implementar el modelo registrado y recibir todos los archivos que se registraron.

No se puede eliminar un modelo registrado que esté usando una implementación activa.

Para obtener un ejemplo de registro de un modelo, consulte [Entrenamiento de un modelo de clasificación de imágenes con Azure Machine Learning](tutorial-train-models-with-aml.md).

### <a name="environments"></a>Entornos

Los entornos de Azure ML se usan para especificar la configuración (Docker/Python/Spark/etc.) que se usa para crear un entorno reproducible para la preparación de datos, el entrenamiento de modelos y la entrega de modelos. Son entidades administradas y con control de versiones en el área de trabajo de Azure Machine Learning que permiten flujos de trabajo de aprendizaje automático reproducibles, auditables y portátiles en distintos destinos de proceso.

Puede usar un objeto de entorno en el proceso local para desarrollar el script de entrenamiento, volver a usar ese mismo entorno en Proceso de Machine Learning para el entrenamiento de modelos a escala e incluso implementar el modelo con ese mismo entorno. 

Descubra [cómo crear y administrar un entorno de ML reutilizable](how-to-use-environments.md) para el entrenamiento y la inferencia.

### <a name="training-scripts"></a>Scripts de entrenamiento

Para entrenar un modelo, especifique el directorio que contiene el script de entrenamiento y los archivos asociados. También puede especificar un nombre del experimento que se usará para almacenar la información recopilada durante el entrenamiento. Durante dicho proceso de entrenamiento, se copia todo el directorio en el entorno de entrenamiento (destino de proceso) y se inicia el script que especifique la configuración de ejecución. También se almacena una instantánea del directorio en el experimento del área de trabajo.

Para obtener un ejemplo, vea [Tutorial: Entrenamiento de un modelo de clasificación de imágenes con Azure Machine Learning](tutorial-train-models-with-aml.md).

### <a name="estimators"></a>Estimadores

Para facilitar el entrenamiento de modelos con marcos conocidos, la clase Estimator le permite construir fácilmente configuraciones de ejecución. Puede crear y usar un objeto [Estimator](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.estimator?view=azure-ml-py) genérico para enviar scripts de entrenamiento que usen cualquier plataforma de aprendizaje que elija, como scikit-learn.

En las tareas de PyTorch, TensorFlow y Chainer, Azure Machine Learning también proporciona los objetos Estimator de [PyTorch](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.pytorch?view=azure-ml-py), [TensorFlow](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py) y [Chainer](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.chainer?view=azure-ml-py) para simplificar el uso de estas plataformas.

Para más información, consulte los siguientes artículos.

* [Entrenamiento de modelos de aprendizaje automático con estimadores](how-to-train-ml-models.md).
* [Entrenamiento de modelos de aprendizaje profundo de PyTorch a escala con Azure Machine Learning](how-to-train-pytorch.md).
* [Entrenamiento y registro de modelos de TensorFlow a escala con Azure Machine Learning](how-to-train-tensorflow.md).
* [Entrenamiento y registro de modelos de Chainer a escala con Azure Machine Learning](how-to-train-ml-models.md).

### <a name="endpoints"></a>Puntos de conexión

Un punto de conexión es la creación de una instancia del modelo en un servicio web que puede hospedarse en la nube o un módulo de IoT para las implementaciones de dispositivos integrados.

#### <a name="web-service-endpoint"></a>Punto de conexión de servicio web

Al implementar un modelo como un servicio web, el punto de conexión se puede implementar en Azure Container Instances, Azure Kubernetes Service o FPGA. El servicio se crea a partir del modelo, el script y los archivos asociados. Estos se colocan en una imagen de contenedor base que contiene el entorno de ejecución para el modelo. La imagen tiene un punto de conexión HTTP de carga equilibrada que recibe solicitudes de puntuación que se envían al servicio web.

Azure le ayuda a supervisar el servicio web mediante la recopilación de telemetría de Application Insight o telemetría de modelos, si es que ha optado por habilitar esta característica. Solo usted puede obtener acceso a los datos de telemetría, que se almacenan en las instancias de su cuenta de almacenamiento y Application Insights.

Si ha habilitado el ajuste automático de escala, Azure ajustará automáticamente la escala de su implementación.

Para obtener un ejemplo de implementación de un modelo como servicio web, consulte [Implementación de un modelo de clasificación de imágenes en Azure Container Instances](tutorial-deploy-models-with-aml.md).

#### <a name="iot-module-endpoints"></a>Puntos de conexión del módulo de IoT

Un punto de conexión del módulo de IoT implementado es un contenedor de Docker que incluye el modelo y el script asociado o la aplicación y las dependencias adicionales. Estos módulos se implementan con Azure IoT Edge en dispositivos Edge.

Si ha habilitado la supervisión, Azure recopila datos de telemetría desde el modelo que está en el módulo de Azure IoT Edge. Solo usted puede obtener acceso a los datos de telemetría, que se almacenan en las instancias de su cuenta de almacenamiento.

Azure IoT Edge garantiza que el módulo se esté ejecutando y supervisa el dispositivo que lo hospeda.


### <a name="compute-instance"></a><a name="compute-instance"></a>Instancia de proceso

Una **instancia de proceso de Azure Machine Learning** (anteriormente máquina virtual de cuadernos) es una estación de trabajo basada en la nube totalmente administrada que incluye varias herramientas y entornos instalados para el aprendizaje automático. Las instancias de proceso se pueden usar como destino de proceso para los trabajos de entrenamiento e inferencia. En el caso de tareas de gran tamaño, los [clústeres de procesos de Azure Machine Learning](how-to-set-up-training-targets.md#amlcompute) con las funcionalidades de escalado de varios nodos es una mejor opción de destino de proceso.

Más información sobre las [instancias de procesos](concept-compute-instance.md).

### <a name="datasets-and-datastores"></a>Conjuntos de datos y almacenes de datos

**Conjuntos de datos de Azure Machine Learning** (versión preliminar) hace que sea más sencillo acceder a los datos y trabajar con ellos. Los conjuntos de datos administran datos en varios escenarios, como el entrenamiento de modelos y la creación de canalizaciones. Con el SDK de Azure Machine Learning, puede acceder al almacenamiento subyacente, explorar los datos y administrar el ciclo de vida de las distintas definiciones de conjuntos de datos.

Los conjuntos de datos proporcionan métodos para trabajar con datos en formatos populares, como el uso de `from_delimited_files()` o `to_pandas_dataframe()`.

Para obtener más información, consulte [Creación y registro de los conjuntos de datos de Azure Machine Learning](how-to-create-register-datasets.md).  Para ver más ejemplos del uso de los conjuntos de datos, consulte los [cuadernos de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/datasets-tutorial).

Un **almacén de datos** es una abstracción de almacenamiento en una cuenta de Azure Storage. El almacén de datos puede usar un contenedor de blobs de Azure o un recurso compartido de archivos de Azure como almacenamiento back-end. Cada área de trabajo tiene un almacén de datos predeterminado en el que puede registrar almacenes de datos adicionales. Use la API del SDK de Python o la CLI de Azure Machine Learning para almacenar y recuperar archivos desde el almacén de datos.

### <a name="compute-targets"></a>Destinos de proceso

Un [destino de proceso](concept-compute-target.md) le permite especificar el recurso de proceso en el que se ejecuta el script de entrenamiento o se hospeda la implementación del servicio web. Esta ubicación puede ser su equipo local o un recurso de proceso en la nube.

Obtenga más información sobre los [destinos de proceso disponibles para el entrenamiento y la implementación](concept-compute-target.md).

### <a name="next-steps"></a>Pasos siguientes

Para una introducción a Azure Machine Learning, consulte:

* [¿Qué es Azure Machine Learning?](overview-what-is-azure-ml.md)
* [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md)
* [Tutorial (parte 1): Entrenamiento de un modelo](tutorial-train-models-with-aml.md)
