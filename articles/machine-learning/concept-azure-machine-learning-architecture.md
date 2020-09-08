---
title: Arquitectura y conceptos clave
titleSuffix: Azure Machine Learning
description: Obtenga información sobre la arquitectura, la terminología y los conceptos que conforman Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sgilley
author: sdgilley
ms.date: 08/20/2020
ms.custom: seoapril2019, seodec18
ms.openlocfilehash: d7bad24510f74a7fadd74328e24ea22855e6fe02
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88750865"
---
# <a name="how-azure-machine-learning-works-architecture-and-concepts"></a>Funcionamiento de Azure Machine Learning: Arquitectura y conceptos

Obtenga información sobre la arquitectura y los conceptos de Azure Machine Learning.

> [!NOTE]
> Aunque en este artículo se definen los términos y conceptos que usa Azure Machine Learning, no se definen los términos y conceptos de la plataforma Azure. Para obtener más información sobre la terminología de la plataforma Azure, consulte el [glosario de Microsoft Azure](https://docs.microsoft.com/azure/azure-glossary-cloud-terminology).

## <a name="workspace"></a><a name="workspace"></a> Área de trabajo

:::image type="content" source="media/concept-azure-machine-learning-architecture/architecture.svg" alt-text="Arquitectura de Azure Machine Learning":::

Un [área de trabajo de aprendizaje automático](concept-workspace.md) es el recurso de nivel superior de Azure Machine Learning.  El área de trabajo es el lugar centralizado para:
* Administrar los recursos que se usan para entrenar e implementar modelos, como [procesos](#compute-instance)
* Almacenar los recursos creados al usar Azure Machine Learning, incluidos:
  * [Entornos](#environments)
  * [Ejecuciones](#runs)
  * [Canalizaciones](#ml-pipelines)
  * [Conjuntos de datos](#datasets-and-datastores)
  * [Models](#models)
  * [Extremos](#endpoints)

Un área de trabajo incluye otros recursos de Azure que se usan en el área de trabajo:

+ [Azure Container Registry](https://azure.microsoft.com/services/container-registry/): registra los contenedores de docker que usa durante el entrenamiento y al implementar un modelo. Para reducir los costos, ACR se **carga de forma diferida** hasta que se crean imágenes de la implementación.
+ [Cuenta de Azure Storage](https://azure.microsoft.com/services/storage/): se usa como almacén de datos predeterminado para el área de trabajo.  Las instancias de Jupyter Notebook que se usan con las instancias de proceso de Azure Machine Learning también se almacenan aquí.
+ [Azure Application Insights](https://azure.microsoft.com/services/application-insights/): almacena información sobre la supervisión de los modelos.
+ [Azure Key Vault](https://azure.microsoft.com/services/key-vault/): almacena secretos que usan los destinos de proceso y otra información confidencial que el área de trabajo necesita.

Puede compartir un área de trabajo con otros usuarios.

## <a name="studio"></a>Estudio

[Azure Machine Learning Studio](https://ml.azure.com) proporciona una vista web de todos los artefactos del área de trabajo.  Este portal también es el lugar desde donde se puede acceder a las herramientas interactivas que forman parte de Azure Machine Learning:

+ [Diseñador de Azure Machine Learning (versión preliminar)](concept-designer.md) para realizar los pasos del flujo de trabajo sin escribir código
+ Experiencia web para el [aprendizaje automático automatizado](concept-automated-ml.md)
+ [Proyectos de etiquetado de datos](how-to-create-labeling-projects.md) para crear, administrar y supervisar proyectos para etiquetar los datos

##  <a name="computes"></a>Procesos

<a name="compute-targets"></a> Un [destino de proceso](concept-compute-target.md) es el equipo o conjunto de equipos donde se ejecuta el script de entrenamiento o se hospeda la implementación del servicio. Esta ubicación puede ser su equipo local o un recurso de proceso remoto.

Azure Machine Learning presenta dos recursos de proceso basados en la nube totalmente administrados que se configuran para tareas de aprendizaje automático:

* <a name="compute-instance"></a> **Instancia de proceso** ([computeinstance](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.computeinstance?view=azure-ml-py)): una instancia de proceso es una máquina virtual (VM) que incluye varias herramientas y entornos instalados para el aprendizaje automático. Use una instancia de proceso como estación de trabajo de desarrollo para empezar a ejecutar cuadernos de ejemplo sin necesidad de instalación. También se puede usar como destino de proceso para los trabajos de entrenamiento e inferencia.
* **Clústeres de proceso** ([amlcompute](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute?view=azure-ml-py)): clúster de máquinas virtuales con funcionalidades de escalado de varios nodos. Se escala verticalmente de manera automática cuando se envía un trabajo. Es más conveniente para los destinos de proceso de producción y trabajos grandes. Úselo como destino de proceso de entrenamiento o para la implementación de desarrollo/pruebas.

Para más información sobre los destinos de proceso de entrenamiento, vea [Entrenamiento de destinos de proceso](concept-compute-target.md#train).  Para más información sobre los destinos de proceso de implementación, vea [Destinos de implementación](concept-compute-target.md#deploy).

## <a name="datasets-and-datastores"></a>Conjuntos de datos y almacenes de datos

Los [**Conjuntos de datos de Azure Machine Learning**](concept-data.md#datasets) hacen que sea más sencillo acceder a los datos y trabajar con ellos. Los conjuntos de datos administran datos en varios escenarios, como el entrenamiento de modelos y la creación de canalizaciones. Con el SDK de Azure Machine Learning, puede acceder al almacenamiento subyacente, explorar los datos y administrar el ciclo de vida de las distintas definiciones de conjuntos de datos.

Los conjuntos de datos proporcionan métodos para trabajar con datos en formatos populares, como el uso de `from_delimited_files()` o `to_pandas_dataframe()`.

Para obtener más información, consulte [Creación y registro de los conjuntos de datos de Azure Machine Learning](how-to-create-register-datasets.md).  Para ver más ejemplos del uso de los conjuntos de datos, consulte los [cuadernos de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/datasets-tutorial).

Un [**almacén de datos**](concept-data.md#datastores) es una abstracción de almacenamiento en una cuenta de Azure Storage. Cada área de trabajo tiene un almacén de datos predeterminado en el que puede registrar almacenes de datos adicionales. Use la API del SDK de Python o la CLI de Azure Machine Learning para almacenar y recuperar archivos desde el almacén de datos. 

## <a name="models"></a>Modelos

En su forma más simple, un modelo es un fragmento de código que toma una entrada y genera una salida. El hecho de crear un modelo de Machine Learning implica seleccionar un algoritmo, proporcionarle datos y [ajustar los hiperparámetros](how-to-tune-hyperparameters.md). El entrenamiento es un proceso iterativo que genera un modelo entrenado, que encapsula lo que el modelo ha aprendido durante el proceso de aprendizaje.

Un modelo se genera mediante una [ejecución](#runs) de un [experimento](#experiments) en Azure Machine Learning. También puede usar un modelo entrenado fuera de Azure Machine Learning. Después, [registre el modelo](#register-model) en el área de trabajo.

Azure Machine Learning es independiente del marco de trabajo. Al crear un modelo, puede usar cualquier marco de aprendizaje automático popular, como Scikit-learn, XGBoost, PyTorch, TensorFlow y Chainer.

Para ver un ejemplo de cómo entrenar un modelo con Scikit-learn, consulte [Tutorial: Entrenamiento de un modelo de clasificación de imágenes con Azure Machine Learning](tutorial-train-models-with-aml.md).

### <a name="model-registry"></a><a name="register-model"></a> Registro de modelos
[Área de trabajo](#workspace) > **Registro de modelos**

El **registro de modelos** permite realizar un seguimiento de todos los modelos del área de trabajo de Azure Machine Learning.

Los modelos se identifican por el nombre y la versión. Cada vez que registra un modelo con el mismo nombre que uno ya existente, el registro interpreta que se trata de una nueva versión. La versión se incrementa y el nuevo modelo se registra con el nombre en cuestión.

Una vez registrado el modelo, puede proporcionar etiquetas de metadatos adicionales y, después, usar esas etiquetas al buscar modelos.

> [!TIP]
> Un modelo registrado es un contenedor lógico para uno o varios archivos que componen el modelo. Por ejemplo, si tiene un modelo que se almacena en varios archivos, puede registrarlos como un único modelo en el área de trabajo de Azure Machine Learning. Después del registro, puede descargar o implementar el modelo registrado y recibir todos los archivos que se registraron.

No se puede eliminar un modelo registrado que esté usando una implementación activa.

Para obtener un ejemplo de registro de un modelo, consulte [Entrenamiento de un modelo de clasificación de imágenes con Azure Machine Learning](tutorial-train-models-with-aml.md).


### <a name="environments"></a>Entornos

[Área de trabajo](#workspace) > **Entornos**

Un [entorno](concept-environments.md) es la encapsulación del lugar en el que se produce el entrenamiento o la puntuación del modelo de Machine Learning. El entorno especifica los paquetes, las variables de entorno y la configuración de software de Python en torno a los scripts de entrenamiento y puntuación.

Para ver ejemplos de código, consulte la sección "Administración de entornos" de [Uso de entornos](how-to-use-environments.md#manage-environments).

### <a name="experiments"></a>Experimentos

[Área de trabajo](#workspace) > **Experimentos**

Un experimento es una agrupación de varias ejecuciones de un script determinado. Siempre pertenece a un área de trabajo. Cuando envíe una ejecución, proporcione un nombre de experimento. La información de la ejecución se almacena en ese experimento. Si el nombre no existe al enviar un experimento, este último se crea automáticamente.

Para obtener un ejemplo del uso de un experimento, consulte [Tutorial: Entrenamiento del primer modelo](tutorial-1st-experiment-sdk-train.md).

### <a name="runs"></a>Ejecuciones

[Área de trabajo](#workspace) > [Experimentos](#experiments) > **Ejecutar**

Una ejecución hace referencia a una única ejecución de un script de entrenamiento. Un experimento normalmente contendrá varias ejecuciones.

Azure Machine Learning registra todas las ejecuciones y almacena la siguiente información en el experimento:

* Metadatos sobre la ejecución (marca de tiempo, duración, etc.).
* Métricas que el script registra.
* Archivos de salida que recopila automáticamente el experimento o que carga explícitamente usted mismo.
* Instantánea del directorio que contiene los scripts antes de la ejecución.

Una ejecución se produce cuando se envía un script para entrenar un modelo. Una ejecución puede tener cualquier número de ejecuciones secundarias. Por ejemplo, la ejecución de nivel superior puede tener dos ejecuciones secundarias, cada una de las cuales puede tener sus propias ejecuciones secundarias.

### <a name="run-configurations"></a>Configuraciones de ejecución

[Área de trabajo](#workspace) > [Experimentos](#experiments) > [Ejecutar](#runs) > **Configuración de ejecución**

Una configuración de ejecución es un conjunto de instrucciones que define cómo se debe ejecutar un script en un destino de proceso determinado. Esta configuración incluye un amplio conjunto de definiciones de comportamiento como, por ejemplo, si quiere usar un entorno de Python existente o un entorno de Conda creado a partir de la especificación.

Una configuración de ejecución puede conservarse en un archivo dentro del directorio que contiene el script de entrenamiento.   O bien, se puede crear como un objeto en memoria y usarse para enviar una ejecución.

Para ver configuraciones de ejecución de ejemplo, consulte [Selección y uso de un destino de proceso para entrenar el modelo](how-to-set-up-training-targets.md).

### <a name="estimators"></a>Estimadores

Para facilitar el entrenamiento de modelos con marcos conocidos, la clase Estimator le permite construir fácilmente configuraciones de ejecución. Puede crear y usar un objeto [Estimator](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.estimator?view=azure-ml-py) genérico para enviar scripts de entrenamiento que usen cualquier plataforma de aprendizaje que elija, como scikit-learn.

En las tareas de PyTorch, TensorFlow y Chainer, Azure Machine Learning también proporciona los objetos Estimator de [PyTorch](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.pytorch?view=azure-ml-py), [TensorFlow](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py) y [Chainer](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.chainer?view=azure-ml-py) para simplificar el uso de estas plataformas.

Para más información, consulte los siguientes artículos.

* [Entrenamiento de modelos de aprendizaje automático con estimadores](how-to-train-ml-models.md).
* [Entrenamiento de modelos de aprendizaje profundo de PyTorch a escala con Azure Machine Learning](how-to-train-pytorch.md).
* [Entrenamiento y registro de modelos de TensorFlow a escala con Azure Machine Learning](how-to-train-tensorflow.md).
* [Entrenamiento y registro de modelos de Chainer a escala con Azure Machine Learning](how-to-train-ml-models.md).

### <a name="snapshots"></a>Instantáneas

[Área de trabajo](#workspace) > [Experimentos](#experiments) > [Ejecutar](#runs) > **Instantánea**

Al enviar una ejecución, Azure Machine Learning comprime el directorio que contiene el script como un archivo zip y lo envía al destino de proceso. A continuación, el archivo .zip se extrae y el script se ejecuta. Azure Machine Learning también almacena el archivo .zip como una instantánea como parte del registro de ejecución. Cualquier persona con acceso al área de trabajo puede buscar un registro de ejecución y descargar la instantánea.


### <a name="logging"></a>Registro

Al desarrollar la solución, use el SDK de Python de Azure Machine Learning en el script de Python para registrar las métricas arbitrarias. Después de la ejecución, consulte las métricas para determinar si la ejecución ha generado el modelo que quiere implementar.


> [!NOTE]
> [!INCLUDE [amlinclude-info](../../includes/machine-learning-amlignore-gitignore.md)]

### <a name="git-tracking-and-integration"></a>Integración y seguimiento de Git

Cuando se inicia una ejecución de entrenamiento en la que el directorio de origen es un repositorio de GIT local, se almacena información sobre el repositorio en el historial de ejecución. Esto funciona con ejecuciones enviadas mediante un estimador, una canalización de aprendizaje automático o una ejecución del script. También funciona para las ejecuciones enviadas desde el SDK o la CLI de Machine Learning.

Para más información, consulte [Integración de Git con Azure Machine Learning](concept-train-model-git-integration.md).

## <a name="deployment"></a>Implementación

Implemente un [modelo registrado](#register-model) como un punto de conexión de servicio. Necesita los siguientes componentes:

* **Entorno**. Este entorno encapsula las dependencias necesarias para ejecutar el modelo para la inferencia.
* **Código de puntuación**. Este script acepta solicitudes, puntúa las solicitudes mediante el modelo y devuelve los resultados.
* **Configuración de inferencia**. La configuración de inferencia especifica el entorno, el script de entrada y otros componentes necesarios para ejecutar el modelo como un servicio.

Para más información sobre estos componentes, vea [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).

### <a name="endpoints"></a>Puntos de conexión

[Área de trabajo](#workspace) > **Puntos de conexión**

Un punto de conexión es la creación de una instancia del modelo en un servicio web que puede hospedarse en la nube o un módulo de IoT para las implementaciones de dispositivos integrados.

#### <a name="web-service-endpoint"></a>Punto de conexión de servicio web

Al implementar un modelo como un servicio web, el punto de conexión se puede implementar en Azure Container Instances, Azure Kubernetes Service o FPGA. El servicio se crea a partir del modelo, el script y los archivos asociados. Estos se colocan en una imagen de contenedor base que contiene el entorno de ejecución para el modelo. La imagen tiene un punto de conexión HTTP de carga equilibrada que recibe solicitudes de puntuación que se envían al servicio web.

Puede habilitar la telemetría de Application Insights o la telemetría del modelo para supervisar el servicio web. Solo usted puede acceder a los datos de telemetría.  Se almacenan en las instancias de Application Insights y de la cuenta de almacenamiento.

Si ha habilitado el ajuste automático de escala, Azure ajustará automáticamente la escala de su implementación.

Para obtener un ejemplo de implementación de un modelo como servicio web, consulte [Implementación de un modelo de clasificación de imágenes en Azure Container Instances](tutorial-deploy-models-with-aml.md).

#### <a name="iot-module-endpoints"></a>Puntos de conexión del módulo de IoT

Un punto de conexión del módulo de IoT implementado es un contenedor de Docker que incluye el modelo y el script asociado o la aplicación y las dependencias adicionales. Estos módulos se implementan con Azure IoT Edge en dispositivos Edge.

Si ha habilitado la supervisión, Azure recopila datos de telemetría desde el modelo que está en el módulo de Azure IoT Edge. Solo usted puede obtener acceso a los datos de telemetría, que se almacenan en las instancias de su cuenta de almacenamiento.

Azure IoT Edge garantiza que el módulo se esté ejecutando y supervisa el dispositivo que lo hospeda.
. 
## <a name="automation"></a>Automation

### <a name="azure-machine-learning-cli"></a>CLI de Azure Machine Learning 

La [CLI de Azure Machine Learning](reference-azure-machine-learning-cli.md) es una extensión de la CLI de Azure, una interfaz de la línea de comandos multiplataforma para la plataforma Azure. Esta extensión proporciona comandos para automatizar las actividades de aprendizaje automático.

### <a name="ml-pipelines"></a>Canalizaciones de Machine Learning

Las [canalizaciones de aprendizaje automático](concept-ml-pipelines.md) se usan para crear y administrar flujos de trabajo que unen las fases de aprendizaje automático. Por ejemplo, una canalización podría incluir las fases de preparación de los datos, entrenamiento del modelo, implementación del modelo e inferencia y puntuación. Cada fase puede estar formada por varios pasos, cada uno de los cuales puede ejecutarse en modo desatendido en varios destinos de proceso. 

Los pasos de canalización se pueden reutilizar y se pueden ejecutar sin volver a ejecutar los pasos anteriores si la salida de esos pasos no ha cambiado. Por ejemplo, puede volver a entrenar un modelo sin volver a ejecutar los costosos pasos de preparación de datos si los datos no han cambiado. Las canalizaciones también permiten a los científicos de datos colaborar mientras trabajan en áreas independientes de un flujo de trabajo de Machine Learning.

## <a name="interacting-with-machine-learning"></a>Interacción con el aprendizaje automático

> [!IMPORTANT]
> Las herramientas marcadas (versión preliminar) a continuación se encuentran actualmente en versión preliminar pública.
> Se ofrece la versión preliminar sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

+  Interactúe con el servicio en cualquier entorno de Python con el [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py).
+ Interactúe con el servicio en cualquier entorno de R con el [SDK de Azure Machine Learning para R](https://azure.github.io/azureml-sdk-for-r/reference/index.html) (versión preliminar).
+ Use el [diseñador de Azure Machine Learning (versión preliminar)](concept-designer.md) para realizar los pasos del flujo de trabajo sin escribir código. (Para usar el diseñador se necesita un [área de trabajo Enterprise](concept-workspace.md#upgrade)).
+ Use la [CLI de Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/reference-azure-machine-learning-cli) para la automatización.
+ El [Acelerador de soluciones Many Models](https://aka.ms/many-models) (versión preliminar) se basa en Azure Machine Learning y permite entrenar, usar y administrar cientos o incluso miles de modelos de Machine Learning.

## <a name="next-steps"></a>Pasos siguientes

Para una introducción a Azure Machine Learning, consulte:

* [¿Qué es Azure Machine Learning?](overview-what-is-azure-ml.md)
* [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md)
* [Tutorial (parte 1): Entrenamiento de un modelo](tutorial-train-models-with-aml.md)
