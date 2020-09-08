---
title: ¿Qué son las canalizaciones de Azure Machine Learning?
description: Obtenga información sobre cómo las canalizaciones de Machine Learning (ML) ayudan a crear, optimizar y administrar los flujos de trabajo de Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: laobri
author: lobrien
ms.date: 08/17/2020
ms.custom: devx-track-python
ms.openlocfilehash: a62d12aa92e41d4a91f963d962616af11d917195
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88604452"
---
# <a name="what-are-azure-machine-learning-pipelines"></a>¿Qué son las canalizaciones de Azure Machine Learning?

En este artículo, obtenga información sobre cómo las canalizaciones de Azure Machine Learning ayudan a crear, optimizar y administrar los flujos de trabajo de Machine Learning. Estos flujos de trabajo tienen una serie de ventajas: 

+ Simplicidad
+ Velocidad
+ Repetibilidad
+ Flexibilidad
+ Control de versiones y seguimiento
+ Modularidad 
+ Control de calidad
+ Control de costos

Estas ventajas pasan a ser significativas cuando el proyecto de aprendizaje automático va más allá de la exploración pura y pasa a la iteración. Incluso las canalizaciones sencillas de un solo paso pueden ser valiosas. Por lo general, los proyectos de aprendizaje automático se encuentran en un estado complejo, y puede ser un alivio lograr que un flujo de trabajo único sea un proceso trivial.

<a name="compare"></a>
### <a name="which-azure-pipeline-technology-should-i-use"></a>¿Qué tecnología de canalización de Azure se debe usar?

La nube de Azure proporciona otras canalizaciones, cada una con una finalidad diferente. En la tabla siguiente se enumeran las diferentes canalizaciones y para qué se usan:

| Escenario | Rol principal | Oferta de Azure | Oferta de OSS | Canalización canónica | Puntos destacados | 
| -------- | --------------- | -------------- | ------------ | -------------- | --------- | 
| Orquestación de modelo (aprendizaje automático) | Científico de datos | Canalizaciones de Azure Machine Learning | Canalizaciones de Kubeflow | Datos -> Modelo | Distribución, almacenamiento en caché, Code First, reutilización | 
| Orquestación de datos (preparación de datos) | Ingeniero de datos | [Canalizaciones de Azure Data Factory](https://docs.microsoft.com/azure/data-factory/concepts-pipelines-activities) | Airflow de Apache | Datos -> Datos | Actividades centradas en datos y movimiento fuertemente tipado |
| Orquestación de códigos y aplicaciones (CI/CD) | Desarrollo/operaciones de aplicaciones | [Canalizaciones de Azure DevOps](https://azure.microsoft.com/services/devops/pipelines/) | Jenkins | Código + modelo-> Aplicación o servicio | Compatibilidad con la mayoría de las actividades abiertas y flexibles, colas de aprobación, fases con restricción de acceso | 

## <a name="what-can-azure-ml-pipelines-do"></a>¿Qué pueden hacer las canalizaciones de Azure ML?

Una canalización de Azure Machine Learning es un flujo de trabajo ejecutable independiente de una tarea de aprendizaje automático completa. Las subtareas se encapsulan como una serie de pasos en la canalización. Una canalización de Azure Machine Learning puede ser tan simple como una llamada a un script de Python, por lo que _puede_ hacer prácticamente de todo. Las canalizaciones _se deben_ centrar en tareas de aprendizaje automático como:

+ La preparación de datos, incluida la importación, validación y limpieza, manipulación y transformación, normalización y almacenamiento provisional.
+ La configuración del entrenamiento, incluida la parametrización de argumentos, rutas de archivos y configuraciones de registros o informes.
+ Entrenamiento y validación de forma eficaz y repetida. La eficiencia puede provenir de especificar subconjuntos de datos específicos, distintos recursos de proceso de hardware, el procesamiento distribuido y la supervisión del progreso
+ La implementación, incluido el control de versiones, escalado, aprovisionamiento y control de acceso.

Los pasos independientes permiten que varios científicos de datos trabajen en la misma canalización al mismo tiempo sin necesidad de sobrecargar los recursos de proceso. Los pasos independientes también facilitan el uso de distintos tamaños y tipos de proceso para cada paso.

Una vez diseñada la canalización, a menudo se aplican más ajustes en torno al bucle de aprendizaje de la canalización. Cuando se vuelve a ejecutar una canalización, la ejecución salta a los pasos necesarios para volver a ejecutarse, como un script de entrenamiento actualizado. Se omiten los pasos que no es necesario volver a ejecutar. 

Con las canalizaciones, puede elegir usar hardware diferente para distintas tareas. Azure coordina los distintos [destinos de proceso](concept-azure-machine-learning-architecture.md) que se usan para que los datos intermedios fluyan sin problema a los destinos del proceso de bajada.

Puede [realizar un seguimiento de las métricas de los experimentos de canalización](https://docs.microsoft.com/azure/machine-learning/how-to-track-experiments) directamente en Azure Portal o en la [página de aterrizaje del área de trabajo (versión preliminar)](https://ml.azure.com). Una vez que se publica una canalización, puede configurar un punto de conexión REST que le permite volver a ejecutar la canalización desde cualquier plataforma o pila.

En resumen, las canalizaciones pueden ayudar con todas las tareas complejas del ciclo de vida del aprendizaje automático. Otras tecnologías de canalización de Azure tienen sus propias ventajas. Las [canalizaciones de Azure Data Factory](https://docs.microsoft.com/azure/data-factory/concepts-pipelines-activities) destaca en el trabajo con datos y [Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/) es la herramienta correcta para la implementación y la integración continua. Pero si su enfoque es el aprendizaje automático, es probable que las canalizaciones de Azure Machine Learning sean la mejor opción para sus necesidades de flujo de trabajo. 

### <a name="analyzing-dependencies"></a>Análisis de dependencias

Muchos ecosistemas de programación tienen herramientas que organizan las dependencias de recursos, bibliotecas o compilaciones. Por lo general, estas herramientas usan marcas de tiempo de archivo para calcular las dependencias. Cuando se cambia un archivo, solo se actualiza el archivo y sus elementos dependientes (se descargan, recompilan o empaquetan). Las canalizaciones de Azure ML amplían este concepto. Al igual que las herramientas de compilación tradicionales, las canalizaciones calculan las dependencias entre los pasos y solo realizan los recálculos necesarios. 

Sin embargo, el análisis de dependencias de las canalizaciones de Azure ML es más sofisticado que las marcas de tiempo sencillas. Cada paso puede ejecutarse en un entorno de hardware y software diferente. La preparación de los datos puede ser un proceso lento, pero no es necesario ejecutarlo en hardware con GPU eficaces, es posible que algunos pasos requieran software específico del sistema operativo, puede que quiera usar entrenamiento distribuido, etc. 

Azure Machine Learning organiza automáticamente todas las dependencias entre los pasos de la canalización. Esta orquestación podría incluir la rotación de imágenes de Docker, la asociación y desasociación de recursos de proceso y el movimiento de datos entre los pasos de manera coherente y automática.

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

## <a name="building-pipelines-with-the-designer"></a>Creación de canalizaciones con el diseñador

Los desarrolladores que prefieren una superficie de diseño visual pueden usar el diseñador de Azure Machine Learning para crear canalizaciones. Puede acceder a esta herramienta desde la selección de **Diseñador** en la página principal del área de trabajo.  El diseñador le permite arrastrar y colocar pasos en la superficie de diseño. 

Al diseñar las canalizaciones visualmente, las entradas y salidas de un paso se muestran de manera visible. Puede arrastrar y colocar conexiones de datos, lo que le permite comprender y modificar rápidamente el flujo de datos de la canalización.

![Ejemplo del diseñador de Azure Machine Learning](./media/concept-designer/designer-drag-and-drop.gif)

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

## <a name="next-steps"></a>Pasos siguientes

Las canalizaciones de Azure ML son una potente instalación que comienza a entregar valor en las primeras fases del desarrollo. El valor aumenta a medida que crece el equipo y el proyecto. En este artículo se ha explicado cómo se especifican las canalizaciones con el SDK de Azure Machine Learning Python y cómo se organizan en Azure. Ha visto código fuente básico y ha conocido algunas de las clases de `PipelineStep` que están disponibles. Ya debe saber cuándo usar las canalizaciones de Azure ML y cómo las ejecuta Azure. 


+ Aprenda a [crear su primera canalización](how-to-create-your-first-pipeline.md).

+ Aprenda a [ejecutar predicciones por lotes en grandes cantidades de datos](tutorial-pipeline-batch-scoring-classification.md ).

+ Consulte los documentos de referencia del SDK para conocer el [centro de las canalizaciones](https://docs.microsoft.com/python/api/azureml-pipeline-core/?view=azure-ml-py) y los [pasos de canalización](https://docs.microsoft.com/python/api/azureml-pipeline-steps/?view=azure-ml-py).

+ Pruebe los cuadernos de Jupyter Notebook que muestran [canalizaciones de Azure Machine Learning](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines). Aprenda a [ejecutar cuadernos para explorar este servicio](samples-notebooks.md).
