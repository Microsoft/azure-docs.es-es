---
title: ¿Qué son las canalizaciones de Azure Machine Learning?
description: Obtenga información sobre cómo las canalizaciones de Machine Learning (ML) ayudan a crear, optimizar y administrar los flujos de trabajo de aprendizaje automático.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: laobri
author: lobrien
ms.date: 04/01/2020
ms.custom: devx-track-python
ms.openlocfilehash: 441575a33e7c3d54de7b25c06f7a839805f8ac1b
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87875292"
---
# <a name="what-are-azure-machine-learning-pipelines"></a>¿Qué son las canalizaciones de Azure Machine Learning?

En este artículo, puede obtener información sobre cómo las canalizaciones de Machine Learning ayudan a crear, optimizar y administrar los flujos de trabajo de Machine Learning. Estos flujos de trabajo tienen una serie de ventajas: 

+ Simplicidad
+ Velocidad
+ Repetibilidad
+ Flexibilidad
+ Control de versiones y seguimiento
+ Modularidad 
+ Control de calidad
+ Control de costos

Estas ventajas pasan a ser significativas cuando el proyecto de aprendizaje automático va más allá de la exploración pura y pasa a la iteración. Incluso las canalizaciones sencillas de un solo paso pueden ser valiosas. Por lo general, los proyectos de aprendizaje automático se encuentran en un estado complejo, y puede ser un alivio lograr que un flujo de trabajo único sea un proceso trivial.

Aprenda a [crear su primera canalización](how-to-create-your-first-pipeline.md).

![Canalizaciones de aprendizaje automático en Azure Machine Learning](./media/concept-ml-pipelines/pipeline-flow.png)

<a name="compare"></a>
### <a name="which-azure-pipeline-technology-should-i-use"></a>¿Qué tecnología de canalización de Azure se debe usar?

La nube de Azure proporciona otras canalizaciones, cada una con una finalidad diferente. En la tabla siguiente se enumeran las diferentes canalizaciones y para qué se usan:

| Escenario | Rol principal | Oferta de Azure | Oferta de OSS | Canalización canónica | Puntos destacados | 
| -------- | --------------- | -------------- | ------------ | -------------- | --------- | 
| Orquestación de modelo (aprendizaje automático) | Científico de datos | Canalizaciones de Azure Machine Learning | Canalizaciones de Kubeflow | Datos -> Modelo | Distribución, almacenamiento en caché, Code First, reutilización | 
| Orquestación de datos (preparación de datos) | Ingeniero de datos | [Canalizaciones de Azure Data Factory](https://docs.microsoft.com/azure/data-factory/concepts-pipelines-activities) | Airflow de Apache | Datos -> Datos | Movimiento fuertemente tipado. Actividades centradas en datos. |
| Orquestación de códigos y aplicaciones (CI/CD) | Desarrollo/operaciones de aplicaciones | [Canalizaciones de Azure DevOps](https://azure.microsoft.com/services/devops/pipelines/) | Jenkins | Código + modelo-> Aplicación o servicio | Compatibilidad con la mayoría de las actividades abiertas y flexibles, colas de aprobación, fases con restricción de acceso | 

## <a name="what-can-azure-ml-pipelines-do"></a>¿Qué pueden hacer las canalizaciones de Azure ML?

Una canalización de Azure Machine Learning es un flujo de trabajo ejecutable independiente de una tarea de aprendizaje automático completa. Las subtareas se encapsulan como una serie de pasos en la canalización. Una canalización de Azure Machine Learning puede ser tan simple como una llamada a un script de Python, por lo que _puede_ hacer prácticamente de todo. Las canalizaciones _se deben_ centrar en tareas de aprendizaje automático como:

+ La preparación de datos, incluida la importación, validación y limpieza, manipulación y transformación, normalización y almacenamiento provisional.
+ La configuración del entrenamiento, incluida la parametrización de argumentos, rutas de archivos y configuraciones de registros o informes.
+ Entrenamiento y validación de forma eficaz y repetida. La eficiencia puede provenir de especificar subconjuntos de datos específicos, distintos recursos de proceso de hardware, el procesamiento distribuido y la supervisión del progreso
+ La implementación, incluido el control de versiones, escalado, aprovisionamiento y control de acceso. 

Los pasos independientes permiten que varios científicos de datos trabajen en la misma canalización al mismo tiempo sin necesidad de sobrecargar los recursos de proceso. Los pasos independientes también facilitan el uso de distintos tamaños y tipos de proceso para cada paso.

Una vez diseñada la canalización, a menudo se aplican más ajustes en torno al bucle de aprendizaje de la canalización. Cuando se vuelve a ejecutar una canalización, la ejecución salta a los pasos necesarios para volver a ejecutarse, como un script de entrenamiento actualizado. Se omiten los pasos que no es necesario volver a ejecutar. El mismo análisis se aplica a los scripts sin cambios que se usan para realizar el paso. Esta funcionalidad de reutilización ayuda a evitar la ejecución de pasos costosos y lentos, como la ingesta de datos y su transformación si los datos subyacentes no han cambiado.

Con Azure Machine Learning, puede usar diversos kits de herramientas y marcos, como PyTorch o TensorFlow, en cada paso de la canalización. Azure coordina los distintos [destinos de proceso](concept-azure-machine-learning-architecture.md) que usa para que los datos intermedios se puedan compartir con los destinos de proceso de bajada.

Puede [realizar un seguimiento de las métricas de los experimentos de canalización](https://docs.microsoft.com/azure/machine-learning/how-to-track-experiments) directamente en Azure Portal o en la [página de aterrizaje del área de trabajo (versión preliminar)](https://ml.azure.com). Una vez que se publica una canalización, puede configurar un punto de conexión REST que le permite volver a ejecutar la canalización desde cualquier plataforma o pila.

En resumen, las canalizaciones pueden ayudar con todas las tareas complejas del ciclo de vida del aprendizaje automático. Otras tecnologías de canalización de Azure tienen sus propias ventajas. Las [canalizaciones de Azure Data Factory](https://docs.microsoft.com/azure/data-factory/concepts-pipelines-activities) destaca en el trabajo con datos y [Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/) es la herramienta correcta para la implementación y la integración continua. Pero si su enfoque es el aprendizaje automático, es probable que las canalizaciones de Azure Machine Learning sean la mejor opción para sus necesidades de flujo de trabajo. 

## <a name="what-are-azure-ml-pipelines"></a>¿Qué son las canalizaciones de Azure ML?

Una canalización de Azure ML realiza un flujo de trabajo lógico completo con una secuencia de pasos ordenada. Cada paso es una acción de procesamiento discreto. Las canalizaciones se ejecutan en el contexto de un [experimento](https://docs.microsoft.com/python/api/azureml-core/azureml.core.experiment.experiment?view=azure-ml-py) de Azure Machine Learning.

En las primeras fases de un proyecto de ML, está bien tener un solo cuaderno de Jupyter o script de Python que realice todo el trabajo del área de trabajo de Azure y la configuración de recursos, la preparación de datos, la configuración de ejecución, el entrenamiento y la validación. Sin embargo, al igual que las funciones y las clases son la opción preferible a un único bloque de código imperativo, los flujos de trabajo de ML se vuelven la opción preferible a un script o cuaderno monolítico. 

Mediante la modularización de tareas de ML, las canalizaciones respaldan el imperativo de la informática que indica que un componente debe "hacer (solo) una cosa bien hecha". La modularidad es claramente fundamental para el éxito del proyecto cuando se programa en equipos, pero incluso cuando se trabaja por sí solo, incluso un proyecto de ML pequeño implica tareas independientes, donde cada una de las cuales tiene algo de complejidad. Entre las tareas se incluyen la configuración del área de trabajo y el acceso a datos, la preparación de datos, la definición y configuración del modelo y la implementación. Mientras que las salidas de una o varias tareas forman las entradas a otra, los detalles exactos de la implementación de una tarea son, en el mejor de los casos, distracciones irrelevantes en la siguiente. En el peor escenario, el estado de cálculo de una tarea puede generar un error en otra. 

### <a name="analyzing-dependencies"></a>Análisis de dependencias

Muchos ecosistemas de programación tienen herramientas que organizan las dependencias de recursos, bibliotecas o compilaciones. Por lo general, estas herramientas usan marcas de tiempo de archivo para calcular las dependencias. Cuando se cambia un archivo, solo se actualiza el archivo y sus elementos dependientes (se descargan, recompilan o empaquetan). Las canalizaciones de Azure ML amplían este concepto de manera considerable. Al igual que las herramientas de compilación tradicionales, las canalizaciones calculan las dependencias entre los pasos y solo realizan los recálculos necesarios. 

Sin embargo, el análisis de dependencias de las canalizaciones de Azure ML es más sofisticado que las marcas de tiempo sencillas. Cada paso puede ejecutarse en un entorno de hardware y software diferente. La preparación de los datos puede ser un proceso lento, pero no es necesario ejecutarlo en hardware con GPU eficaces, es posible que algunos pasos requieran software específico del sistema operativo, puede que quiera usar entrenamiento distribuido, etc. Si bien los ahorros de costos de optimización de recursos pueden ser importantes, puede que resulte abrumador compatibilizar todas las variaciones en los recursos de hardware y software. Incluso es más difícil hacer todo eso sin cometer ningún error en los datos que se transfieren entre los pasos. 

Las canalizaciones solucionan este problema. Azure Machine Learning organiza automáticamente todas las dependencias entre los pasos de la canalización. Esta orquestación podría incluir la rotación de imágenes de Docker, la asociación y desasociación de recursos de proceso y el movimiento de datos entre los pasos de manera coherente y automática.

### <a name="reusing-results"></a>Reutilización de los resultados

Además, si así lo desea, puede reutilizar la salida de un paso. Si especifica la reutilización como una posibilidad y no hay dependencias ascendentes que desencadenen el recálculo, el servicio de canalización usará una versión almacenada en caché de los resultados del paso. Esta reutilización puede disminuir considerablemente el tiempo de desarrollo. Si tiene una tarea de preparación de datos compleja, es probable que la vuelva a ejecutar más a menudo que lo estrictamente necesario. Las canalizaciones le permiten liberarse de esa preocupación: si es necesario, se ejecutará el paso. Si no lo es, no lo hará.

Todo este análisis, orquestación y activación de dependencia se administran mediante Azure Machine Learning cuando se crea una instancia de un objeto [Pipeline ](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipeline(class)?view=azure-ml-py), se pasa a un `Experiment` y se llama a `submit()`. 

### <a name="coordinating-the-steps-involved"></a>Coordinación de los pasos implicados

Al crear y ejecutar un objeto `Pipeline`, se producen los siguientes pasos de alto nivel:

+ En cada paso, el servicio calcula los requisitos de:
    + Recursos de proceso de hardware
    + Recursos del sistema operativo (imágenes de Docker)
    + Recursos de software (dependencias de Conda/virtualenv)
    + Entradas de datos 
+ El servicio determina las dependencias entre los pasos, lo que da como resultado un grafo de ejecución dinámico.
+ Cuando se ejecuta cada nodo del grafo de ejecución:
    + El servicio configura el entorno de hardware y software necesario (quizás reutilizando los recursos existentes).
    + El paso se ejecuta, lo que proporciona información de registro y supervisión al objeto `Experiment` que lo contiene.
    + Cuando se completa el paso, sus salidas se preparan como entradas al paso siguiente o se escriben en el almacenamiento.
    + Los recursos que ya no son necesarios se finalizan y desasocian.

![Pasos de la canalización](./media/concept-ml-pipelines/run_an_experiment_as_a_pipeline.png)

## <a name="building-pipelines-with-the-python-sdk"></a>Creación de canalizaciones con el SDK de Python

En el [SDK de Python para Azure Machine Learning ](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py), una canalización es un objeto de Python definido en el módulo de `azureml.pipeline.core`. Un objeto [Pipeline](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipeline%28class%29?view=azure-ml-py) contiene una secuencia ordenada de uno o varios objetos [PipelineStep](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.builder.pipelinestep?view=azure-ml-py). La clase `PipelineStep` es abstracta y los pasos reales serán de subclases como [EstimatorStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.estimatorstep?view=azure-ml-py), [PythonScriptStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.pythonscriptstep?view=azure-ml-py) o [DataTransferStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.datatransferstep?view=azure-ml-py). La clase [ModuleStep ](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.modulestep?view=azure-ml-py) contiene una secuencia reutilizable de pasos que se pueden compartir entre las canalizaciones. Un `Pipeline` se ejecuta como parte de un `Experiment`.

Una canalización de Azure ML está asociada a un área de trabajo de Azure Machine Learning y un paso de la canalización está asociado a un destino de proceso disponible dentro de esa área de trabajo. Para más información, consulte [Creación y administración de áreas de trabajo de Azure Machine Learning en Azure Portal](https://docs.microsoft.com/azure/machine-learning/how-to-manage-workspace) o [¿Qué son los destinos de proceso en Azure Machine Learning?](https://docs.microsoft.com/azure/machine-learning/concept-compute-target)

En Azure Machine Learning, un destino de proceso es el entorno en el que se produce una fase de ML. El entorno de software puede ser una máquina virtual remota, un Proceso de Azure Machine Learning, Azure Databricks, Azure Batch, etc. El entorno de hardware también puede variar en gran medida, en función de la compatibilidad con la GPU, la memoria, el almacenamiento, etc. Puede especificar el destino de proceso para cada paso, lo que le proporciona un control exhaustivo sobre los costos. Puede usar recursos más o menos eficaces para las necesidades de acción, volumen de datos y rendimiento específicas del proyecto. 

## <a name="building-pipelines-with-the-designer"></a>Creación de canalizaciones con el diseñador

Los desarrolladores que prefieren una superficie de diseño visual pueden usar el diseñador de Azure Machine Learning para crear canalizaciones. Puede acceder a esta herramienta desde la selección de **Diseñador** en la página principal del área de trabajo.  El diseñador le permite arrastrar y colocar pasos en la superficie de diseño. Para un desarrollo rápido, puede usar los módulos existentes en el espectro de tareas de ML. Los módulos existentes cubren todos los escenarios, desde la transformación de datos hasta la selección de algoritmos y el entrenamiento de la implementación. O bien, puede crear una canalización totalmente personalizada mediante la combinación de sus propios pasos definidos en los scripts de Python.

Al diseñar las canalizaciones visualmente, las entradas y salidas de un paso se muestran de manera visible. Puede arrastrar y colocar conexiones de datos, lo que le permite comprender y modificar rápidamente el flujo de datos de la canalización.
 
![Ejemplo del diseñador de Azure Machine Learning](./media/concept-designer/designer-drag-and-drop.gif)

### <a name="understanding-the-execution-graph"></a>Descripción del grafo de ejecución

Los pasos dentro de una canalización pueden tener dependencias en otros pasos. El servicio de canalización de Azure ML realiza el trabajo de análisis y orquestación de estas dependencias. Los nodos del "grafo de ejecución" resultante son los pasos del procesamiento. Cada paso puede implicar la creación o reutilización de una combinación determinada de hardware y software, la reutilización de los resultados almacenados en caché, etc. La orquestación y optimización del servicio de este grafo de ejecución puede acelerar considerablemente una fase de ML y reducir los costos. 

Dado que los pasos se ejecutan de forma independiente, los objetos que contienen los datos de entrada y salida que fluyen entre los pasos se deben definir de manera externa. Este es el rol de los objetos [DataSet](https://docs.microsoft.com/python/api/azureml-core/azureml.data.data_reference.datareference?view=azure-ml-py) y [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py). Estos objetos de datos están asociados a un objeto [Datastore](https://docs.microsoft.com/python/api/azureml-core/azureml.core.datastore%28class%29?view=azure-ml-py) que encapsula su configuración de almacenamiento. La clase base `PipelineStep` siempre se crea con una cadena `name`, una lista de `inputs` y una lista de `outputs`. Por lo general, también tiene una lista de `arguments` y, a menudo, tendrá una lista de `resource_inputs`. Las subclases suelen tener también argumentos adicionales (por ejemplo, `PythonScriptStep` requiere el nombre de archivo y la ruta de acceso del script que se va a ejecutar). 

El grafo de ejecución es acíclico, pero las canalizaciones se pueden ejecutar según una programación periódica y pueden ejecutar scripts de Python que pueden escribir información de estado en el sistema de archivos, lo que permite crear perfiles complejos. Si diseña la canalización para que determinados pasos se ejecuten en paralelo o de manera asincrónica, Azure Machine Learning controla de forma transparente el análisis de dependencias y la coordinación de la distribución ramificada de salida y entrada. Por lo general, no tiene que preocuparse por los detalles del grafo de ejecución, pero puede encontrarlo a través del atributo [Pipeline.graph](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipeline.pipeline?view=azure-ml-py#attributes). 


### <a name="a-simple-python-pipeline"></a>Una canalización de Python sencilla

Este fragmento de código muestra los objetos y las llamadas necesarios para crear y ejecutar un `Pipeline` básico:

```python
ws = Workspace.from_config() 
blob_store = Datastore(ws, "workspaceblobstore")
compute_target = ws.compute_targets["STANDARD_NC6"]
experiment = Experiment(ws, 'MyExperiment') 

input_data = Dataset.File.from_files(
    DataPath(datastore, '20newsgroups/20news.pkl'))

output_data = PipelineData("output_data", datastore=blob_store)

input_named = input_data.as_named_input('input')

steps = [ PythonScriptStep(
    script_name="train.py",
    arguments=["--input", input_named.as_download(), "--output", output_data],
    inputs=[input_data],
    outputs=[output_data],
    compute_target=compute_target,
    source_directory="myfolder"
) ]

pipeline = Pipeline(workspace=ws, steps=steps)

pipeline_run = experiment.submit(pipeline)
pipeline_run.wait_for_completion()
```

El fragmento de código se inicia con objetos comunes de Azure Machine Learning, un `Workspace`, a `Datastore`, un [ComputeTarget](https://docs.microsoft.com/python/api/azureml-core/azureml.core.computetarget?view=azure-ml-py) y un `Experiment`. Luego, el código crea los objetos para contener `input_data` y `output_data`. La matriz `steps` contiene un elemento único, un `PythonScriptStep` que utilizará los objetos de datos y se ejecutará en el `compute_target`. A continuación, el código crea una instancia del propio objeto `Pipeline` y pasa la matriz de pasos y el área de trabajo. La llamada a `experiment.submit(pipeline)` inicia la ejecución de la canalización de Azure ML. La llamada a `wait_for_completion()` se bloquea hasta que finaliza la canalización. 

Para más información sobre cómo conectar la canalización con los datos, consulte los artículos [Acceso a los datos en Azure Machine Learning](concept-data.md) y [Movimiento de datos a los pasos de canalización de Machine Learning (Python) y entre ellos](how-to-move-data-in-out-of-pipelines.md). 

## <a name="best-practices-when-using-pipelines"></a>Procedimientos recomendados al usar canalizaciones

Como puede ver, la creación de una canalización de Azure ML es un poco más compleja que el inicio de un script. Las canalizaciones requieren la configuración y la creación de algunos objetos de Python. 

Algunas situaciones que sugieren el uso de una canalización:

* En un entorno de equipo: divida las tareas de ML en varios pasos separados para que los desarrolladores puedan trabajar y desarrollar sus programas de manera independiente. 

* Cuando esté en la implementación o cerca de ella: concrete la configuración y usa operaciones programadas y controladas por eventos para mantenerse al día con los cambios de datos.

* En las primeras fases de un proyecto de ML o trabajando solo: use canalizaciones para automatizar la compilación. Si ha empezado a preocuparse de volver a crear la configuración y el estado de cálculo antes de implementar una idea nueva, es una señal de que podría considerar el uso de una canalización para automatizar el flujo de trabajo. 

Es fácil entusiasmarse con la reutilización de los resultados almacenados en caché, el control detallado sobre los costos de proceso y el aislamiento de procesos, pero las canalizaciones tienen costos. Algunos antipatrones incluyen:

* El uso de canalizaciones como único medio para separar los problemas. Las funciones, los objetos y los módulos integrados de Python contribuyen considerablemente a evitar un estado de programación confuso. Un paso de canalización es mucho más costoso que una llamada de función.

* Acoplamiento intenso entre los pasos de la canalización. Si la refactorización de un paso dependiente suele requerir la modificación de las salidas de un paso anterior, es probable que los pasos independientes actualmente signifiquen más un costo que una ventaja. Otra pista de que los pasos están demasiado acoplados son los argumentos de un paso que no son datos, sino marcas para controlar el procesamiento. 

* Optimización prematura de los recursos de proceso. Por ejemplo, a menudo hay varias fases para la preparación de datos y a menudo se puede ver "Oh, aquí podría usar `MpiStep` para la programación en paralelo, pero hay un lugar donde podría usar un `PythonScriptStep` con un destino de proceso menos eficaz", etc. Y quizás, a largo plazo, la creación de pasos específicos como este podría merecer la pena, especialmente si existe la posibilidad de usar resultados almacenados en caché en lugar de volver a calcular siempre. Sin embargo, las canalizaciones no pretenden sustituir el módulo `multiprocessing` nativo de Python. 

Hasta que un proyecto llega a una implementación o cerca de esta, las canalizaciones deben ser más generales en lugar de precisas. Si considera que su proyecto de ML incluye _fases_  y una canalización que proporciona un flujo de trabajo completo para desplazarse por una fase determinada, está en la ruta de acceso correcta. 

## <a name="key-advantages"></a>Ventajas clave

Las ventajas clave de usar canalizaciones para los flujos de trabajo de aprendizaje automático son:

|Ventaja clave|Descripción|
|:-------:|-----------|
|**Ejecuciones&nbsp;desatendidas**|Programe los pasos para la ejecución en paralelo o en secuencia de manera confiable y desatendida. La preparación de datos y su modelado pueden durar días o semanas y las canalizaciones le permiten centrarse en otras tareas mientras se ejecuta el proceso. |
|**Proceso heterogéneo**|Utilice varias canalizaciones coordinadas de forma confiable en ubicaciones de almacenamiento y recursos de procesos heterogéneos y escalables. Use de manera eficaz los recursos de proceso disponibles mediante la ejecución de pasos individuales de la canalización en diferentes destinos de proceso, como HDInsight, instancias de Data Science VM de la GPU y Databricks.|
|**Reusabilidad**|Cree plantillas de canalizaciones para escenarios específicos, como reentrenamiento y puntuación por lotes. Desencadene las canalizaciones publicadas desde sistemas externos a través de llamadas REST sencillas.|
|**Seguimiento y control de versiones**|En lugar de hacer un seguimiento manual de los datos y los resultados conforme itera, use el SDK de canalizaciones para asignar de forma explícita un nombre y una versión a los orígenes de datos, las entradas y las salidas. También puede administrar scripts y datos por separado para aumentar la productividad.|
| **Modularidad** | Separar las áreas de preocupación y aislar los cambios permite que el software evolucione más rápidamente con una mayor calidad. | 
|**Colaboración**|Las canalizaciones permiten que los científicos de datos colaboren en todas las áreas del proceso de diseño de aprendizaje automático, a la vez que pueden trabajar de manera simultánea en los pasos de la canalización.|

### <a name="choosing-the-proper-pipelinestep-subclass"></a>Elección de la subclase PipelineStep adecuada

`PythonScriptStep` es la subclase más flexible del elemento `PipelineStep` abstracto. Otras subclases, como `EstimatorStep` y `DataTransferStep`, pueden realizar tareas específicas con menos código. Por ejemplo, se puede crear un elemento `EstimatorStep` simplemente pasando un nombre para el paso, un elemento `Estimator` y un destino de proceso. También se pueden invalidar las entradas y salidas, los parámetros de canalización y los argumentos. Para obtener más información, vea [Entrenamiento de modelos con Azure Machine Learning mediante un objeto Estimator](how-to-train-ml-models.md). 

`DataTransferStep` facilita el traslado de datos entre orígenes y receptores de datos. El código para hacer esta transferencia manualmente es sencillo pero repetitivo. Como alternativa, puede simplemente crear un elemento `DataTransferStep` con un nombre, referencias a un origen de datos y a un receptor de datos, y un destino de proceso. El cuaderno [Canalización de Azure Machine Learning con DataTransferStep](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/intro-to-pipelines/aml-pipelines-data-transfer.ipynb) muestra esta flexibilidad.

## <a name="modules"></a>Módulos

Aunque los pasos de canalización permiten la reutilización de los resultados de una ejecución anterior, en muchos casos, se supone que en la construcción del paso los scripts y los archivos dependientes necesarios deben estar disponibles localmente. Si un científico de datos desea basarse en el código existente, con frecuencia se deben clonar los scripts y las dependencias desde un repositorio independiente.

Los módulos son de uso similar a los pasos de canalización, pero proporcionan control de versiones facilitado mediante el área de trabajo, lo que permite la colaboración y la reutilización a gran escala. Los módulos están diseñados para reutilizarse en varias canalizaciones y pueden evolucionar para adaptar un cálculo específico a diferentes casos de uso. Los usuarios pueden realizar las siguientes tareas mediante el área de trabajo, sin usar repositorios externos:

* Creación de módulos y publicación de nuevas versiones de módulos existentes
* Desuso de versiones existentes
* Marcado de las versiones como deshabilitadas para impedir que los consumidores usen esa versión
* Designación de versiones predeterminadas
* Recuperación de módulos por versión del área de trabajo para garantizar que los equipos usen el mismo código

Consulte el [cuaderno](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/intro-to-pipelines/aml-pipelines-how-to-use-modulestep.ipynb) para ver ejemplos de código sobre cómo crear, conectar y usar módulos en canalizaciones de Azure Machine Learning.

## <a name="next-steps"></a>Pasos siguientes

Las canalizaciones de Azure ML son una potente instalación que comienza a entregar valor en las primeras fases del desarrollo. El valor aumenta a medida que crece el equipo y el proyecto. En este artículo se ha explicado cómo se especifican las canalizaciones con el SDK de Azure Machine Learning Python y cómo se organizan en Azure. Ha visto código fuente básico y ha conocido algunas de las clases de `PipelineStep` que están disponibles. Ya debe saber cuándo usar las canalizaciones de Azure ML y cómo las ejecuta Azure. 


+ Aprenda a [crear su primera canalización](how-to-create-your-first-pipeline.md).

+ Aprenda a [ejecutar predicciones por lotes en grandes cantidades de datos](tutorial-pipeline-batch-scoring-classification.md ).

+ Consulte los documentos de referencia del SDK para conocer el [centro de las canalizaciones](https://docs.microsoft.com/python/api/azureml-pipeline-core/?view=azure-ml-py) y los [pasos de canalización](https://docs.microsoft.com/python/api/azureml-pipeline-steps/?view=azure-ml-py).

+ Pruebe los cuadernos de Jupyter Notebook que muestran [canalizaciones de Azure Machine Learning](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines). Aprenda a [ejecutar cuadernos para explorar este servicio](samples-notebooks.md).
