---
title: Seguimiento de MLflow para experimentos de Machine Learning
titleSuffix: Azure Machine Learning
description: Configure MLflow con Azure Machine Learning para registrar métricas y artefactos de modelos de aprendizaje automático e implementar los modelos de aprendizaje automático como un servicio web.
services: machine-learning
author: shivp950
ms.author: shipatel
ms.service: machine-learning
ms.subservice: core
ms.reviewer: nibaccam
ms.date: 09/08/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 116faae1bc0a93ce2007fcf809a8c96475289036
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90897207"
---
# <a name="track-model-metrics-and-deploy-ml-models-with-mlflow-and-azure-machine-learning-preview"></a>Seguimiento de métricas del modelo e implementación de modelos de aprendizaje automático con MLflow y Azure Machine Learning (versión preliminar)

En este artículo se muestra cómo habilitar el identificador URI de seguimiento y la API de registro de MLflow, que en conjunto se conocen como [MLflow Tracking](https://mlflow.org/docs/latest/quickstart.html#using-the-tracking-api), para conectar experimentos de MLflow y Azure Machine Learning. 

Con compatibilidad nativa de Azure Machine Learning con MLflow, puede:

+ Realizar un seguimiento de las métricas y los artefactos de los experimentos, así como registrarlos, en el [área de trabajo de Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/concept-azure-machine-learning-architecture#workspaces). Si ya usa Seguimiento de MLflow para los experimentos, el área de trabajo proporciona una ubicación centralizada, segura y escalable para almacenar los modelos y las métricas de entrenamiento.

+ Enviar trabajos de aprendizaje con proyectos de MLflow con compatibilidad de back-end con Azure Machine Learning (versión preliminar). Puede enviar trabajos localmente con el seguimiento de Azure Machine Learning o migrar las ejecuciones a la nube, por ejemplo, a través de un [Proceso de Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/how-to-create-attach-compute-sdk#amlcompute).

+ Hacer un seguimiento de modelos y administrarlos en MLflow y el registro de modelos de Azure Machine Learning.

+ Implementar sus experimentos de MLflow como un servicio web Azure Machine Learning. Mediante la implementación como un servicio web, puede aplicar las funcionalidades de detección del desfase de datos y de supervisión de Azure Machine Learning en los modelos de producción. 

[MLflow](https://www.mlflow.org) es una biblioteca de código abierto para administrar el ciclo de vida de los experimentos de aprendizaje automático. MLFlow Tracking es un componente de MLflow que lleva a cabo un registro y un seguimiento de las métricas de ejecución de entrenamiento y de los artefactos del modelo, independientemente del entorno del experimento (localmente en su equipo, en un destino de proceso remoto, en una máquina virtual o en un clúster de Azure Databricks). 

>[!NOTE]
> Como biblioteca de código abierto, MLflow cambia con frecuencia. Como tal, la funcionalidad que se pone a disposición a través de la integración de Azure Machine Learning y MLflow debe considerarse como una vista previa y no es totalmente compatible con Microsoft.

En el siguiente diagrama se ilustra que con Seguimiento de MLflow, se realiza un seguimiento de las métricas de ejecución de un experimento y se almacenan los artefactos del modelo en el área de trabajo de Azure Machine Learning.

![Diagrama de MLflow con Azure Machine Learning](./media/how-to-use-mlflow/mlflow-diagram-track.png)

> [!TIP]
> La información de este documento va destinada principalmente a aquellos científicos de datos y desarrolladores que deseen supervisar el proceso de entrenamiento del modelo. Los administradores que estén interesados en la supervisión del uso de recursos y eventos desde Azure Machine Learning, como cuotas, ejecuciones de entrenamiento completadas o implementaciones de modelos completadas pueden consultar [Supervisión de Azure Machine Learning](monitor-azure-machine-learning.md).

## <a name="compare-mlflow-and-azure-machine-learning-clients"></a>Comparación entre los clientes de MLflow y Azure Machine Learning

 En la tabla siguiente se resumen los distintos clientes que pueden usar Azure Machine Learning y sus respectivas funcionalidades.

 El Seguimiento de MLflow ofrece funciones de registro de métricas y almacenamiento de artefactos que de otra manera solo están disponibles a través del [SDK de Python de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py&preserve-view=true).

| Capacidad | Implementación y seguimiento de MLflow | SDK de Python de Azure Machine Learning |  CLI de Azure Machine Learning | Azure Machine Learning Studio|
|---|---|---|---|---|
| Administración del área de trabajo |   | ✓ | ✓ | ✓ |
| Uso de almacenes de datos  |   | ✓ | ✓ | |
| Métricas de registro      | ✓ | ✓ |   | |
| Carga de artefactos | ✓ | ✓ |   | |
| Visualización de métricas     | ✓ | ✓ | ✓ | ✓ |
| Administración de procesos   |   | ✓ | ✓ | ✓ |
| Implementación de modelos    | ✓ | ✓ | ✓ | ✓ |
|Supervisión del rendimiento del modelo||✓|  |   |
| Detección del desfase de datos |   | ✓ |   | ✓ |

## <a name="prerequisites"></a>Requisitos previos

* [Instale MLflow](https://mlflow.org/docs/latest/quickstart.html).
* [Instale el SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py&preserve-view=true) en el equipo local. El SDK proporciona la conectividad de MLflow para acceder al área de trabajo.
* [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

## <a name="track-local-runs"></a>Seguimiento de ejecuciones locales

El seguimiento de MLflow con Azure Machine Learning le permite almacenar las métricas y los artefactos registrados desde las ejecuciones locales en el área de trabajo de Azure Machine Learning.

Instale el paquete `azureml-mlflow` que va a usar el Seguimiento de MLflow con Azure Machine Learning en los experimentos ejecutados localmente en un editor de código o Jupyter Notebook.

```shell
pip install azureml-mlflow
```

Importe las clases `mlflow` y [`Workspace`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace(class)?view=azure-ml-py&preserve-view=true) para acceder al URI de seguimiento de MLflow y configurar el área de trabajo.

En el código siguiente, el método `get_mlflow_tracking_uri()` asigna un dirección URI de seguimiento única al área de trabajo (`ws`) y `set_tracking_uri()` apunta el URI de seguimiento de MLflow a esa dirección.

```Python
import mlflow
from azureml.core import Workspace

ws = Workspace.from_config()

mlflow.set_tracking_uri(ws.get_mlflow_tracking_uri())
```

>[!NOTE]
>El URI de seguimiento es válido hasta una hora como máximo. Si reinicia el script después de un tiempo de inactividad, use la API get_mlflow_tracking_uri para obtener un nuevo URI.

Establezca el nombre del experimento de MLflow con `set_experiment()` y comience la ejecución de entrenamiento con `start_run()`. Después, use `log_metric()` para activar la API de registro de MLflow y empezar a registrar las métricas de la ejecución de entrenamiento.

```Python
experiment_name = 'experiment_with_mlflow'
mlflow.set_experiment(experiment_name)

with mlflow.start_run():
    mlflow.log_metric('alpha', 0.03)
```

## <a name="track-remote-runs"></a>Seguimiento de ejecuciones remotas

El seguimiento de MLflow con Azure Machine Learning le permite almacenar las métricas y los artefactos registrados desde las ejecuciones remotas en el área de trabajo de Azure Machine Learning.

Las ejecuciones remotas permiten entrenar modelos en procesos más eficaces, como máquinas virtuales habilitadas para GPU o clústeres de Proceso de Machine Learning. Consulte [Uso de destinos de proceso del entrenamiento del modelo](how-to-set-up-training-targets.md) para obtener información sobre las distintas opciones de proceso.

Configure el proceso y el entorno de ejecución de entrenamiento con la clase [`Environment`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment.environment?view=azure-ml-py&preserve-view=true). Incluya paquetes de PIP `mlflow` y `azureml-mlflow` en la sección [`CondaDependencies`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.conda_dependencies.condadependencies?view=azure-ml-py&preserve-view=true) del entorno. Después, construya [`ScriptRunConfig`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.script_run_config.scriptrunconfig?view=azure-ml-py&preserve-view=true) con el proceso remoto como destino de proceso.

```Python
from azureml.core.environment import Environment
from azureml.core.conda_dependencies import CondaDependencies
from azureml.core import ScriptRunConfig

exp = Experiment(workspace = 'my_workspace',
                 name='my_experiment')

mlflow_env = Environment(name='mlflow-env')

cd = CondaDependencies.create(pip_packages=['mlflow', 'azureml-mlflow'])

mlflow_env.python.conda_dependencies = cd

src = ScriptRunConfig(source_directory='./my_script_location', script='my_training_script.py')

src.run_config.target = 'my-remote-compute-compute'
src.run_config.environment = mlflow_env
```

En el script de entrenamiento, importe `mlflow` para usar las API de registro de MLflow y empiece a registrar las métricas de ejecución.

```Python
import mlflow

with mlflow.start_run():
    mlflow.log_metric('example', 1.23)
```

Con esta configuración del proceso y de la ejecución de entrenamiento, use el método `Experiment.submit()` para enviar una ejecución. Este método establece automáticamente el identificador URI de seguimiento de MLflow y dirige el registro de MLflow al área de trabajo.

```Python
run = exp.submit(src)
```

## <a name="train-with-mlflow-projects"></a>Entrenamiento con proyectos de MLflow

Los [proyectos de MLflow](https://mlflow.org/docs/latest/projects.html) permiten organizar y describir el código para permitir que otros científicos de datos (o herramientas automatizadas) lo ejecuten. Los proyectos de MLflow con Azure Machine Learning le permiten realizar un seguimiento de las ejecuciones de entrenamiento y administrarlas en el área de trabajo. 

En este ejemplo se muestra cómo enviar proyectos de MLflow localmente con seguimiento de Azure Machine Learning.

Instale el paquete `azureml-mlflow` que va a usar el seguimiento de MLflow con Azure Machine Learning en los experimentos localmente. Los experimentos pueden ejecutarse a través de una instancia de Jupyter Notebook o un editor de código.

```shell
pip install azureml-mlflow
```

Importe las clases `mlflow` y [`Workspace`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace(class)?view=azure-ml-py) para acceder al URI de seguimiento de MLflow y configurar el área de trabajo.

```Python
import mlflow
from azureml.core import Workspace

ws = Workspace.from_config()

mlflow.set_tracking_uri(ws.get_mlflow_tracking_uri())
```

Establezca el nombre del experimento de MLflow con `set_experiment()` y comience la ejecución de entrenamiento con `start_run()`. Después, use `log_metric()` para activar la API de registro de MLflow y empezar a registrar las métricas de la ejecución de entrenamiento.

```Python
experiment_name = 'experiment-with-mlflow-projects'
mlflow.set_experiment(experiment_name)
```

Cree el objeto de configuración de back-end para almacenar la información necesaria para la integración, por ejemplo, el destino de proceso y el tipo de entorno administrado que se va a usar.

```python
backend_config = {"USE_CONDA": False}
```
Agregue el paquete `azureml-mlflow` como dependencia pip al archivo de configuración del entorno para realizar el seguimiento de las métricas y los artefactos clave en el área de trabajo. 

``` shell
name: mlflow-example
channels:
  - defaults
  - anaconda
  - conda-forge
dependencies:
  - python=3.6
  - scikit-learn=0.19.1
  - pip
  - pip:
    - mlflow
    - azureml-mlflow
```
Envíe la ejecución local y asegúrese de establecer el parámetro `backend = "azureml" `. Con esta configuración, puede enviar ejecuciones localmente y obtener la compatibilidad adicional del seguimiento de salida automático, archivos de registro, instantáneas y errores impresos en el área de trabajo. 

Vea las ejecuciones y métricas en [Azure Machine Learning Studio](overview-what-is-machine-learning-studio.md). 


```python
local_env_run = mlflow.projects.run(uri=".", 
                                    parameters={"alpha":0.3},
                                    backend = "azureml",
                                    use_conda=False,
                                    backend_config = backend_config, 
                                    )

```

## <a name="track-azure-databricks-runs"></a>Seguimiento de las ejecuciones de Azure Databricks

El seguimiento de MLflow con Azure Machine Learning permite almacenar las métricas y los artefactos de las ejecuciones de Azure Databricks en las tres áreas siguientes a la vez: 

* Área de trabajo de Azure Machine Learning
* Área de trabajo de Azure Databricks
* MLflow

Para ejecutar los experimentos de Mlflow con Azure Databricks, primero debe crear un [área de trabajo y un clúster de Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/quickstart-create-databricks-workspace-portal). En su clúster, instale la biblioteca *azureml-mlflow* desde PyPi para asegurarse de que el clúster tenga acceso a las funciones y clases necesarias.

> [!NOTE]
> El paquete de `azureml.core` incluye `azureml-mlflow`. Si ya ha instalado `azureml.core`, puede omitir el paso de instalación de `azureml-mlflow`. 

Una vez hecho esto, importe su cuaderno de experimentos, adjúntelo a su clúster de Azure Databricks y ejecute el experimento. 

### <a name="install-libraries"></a>Instalar bibliotecas

Para instalar bibliotecas en el clúster, vaya a la pestaña **Bibliotecas** y haga clic en **Instalar nuevo**.

 ![mlflow con azure databricks](./media/how-to-use-mlflow/azure-databricks-cluster-libraries.png)

En el campo **Paquete**, escriba azureml-mlflow y, a continuación, haga clic en instalar. Repita este paso según sea necesario para instalar otros paquetes adicionales en el clúster para el experimento.

 ![Instalación de la biblioteca mlflow de Azure DB](./media/how-to-use-mlflow/install-libraries.png)

### <a name="set-up-your-notebook-and-workspace"></a>Configuración del cuaderno y del área de trabajo

Una vez haya configurado el clúster, importe el cuaderno del experimento, ábralo y asóciele el clúster.

El código siguiente debe estar en el cuaderno del experimento. Este código obtiene los detalles de la suscripción a Azure para crear una instancia del área de trabajo. En este código se supone que tiene un grupo de recursos y un área de trabajo de Azure Machine Learning. Si no es así, puede [crearlos](how-to-manage-workspace.md). 

```python
import mlflow
import mlflow.azureml
import azureml.mlflow
import azureml.core

from azureml.core import Workspace
from azureml.mlflow import get_portal_url

subscription_id = 'subscription_id'

# Azure Machine Learning resource group NOT the managed resource group
resource_group = 'resource_group_name' 

#Azure Machine Learning workspace name, NOT Azure Databricks workspace
workspace_name = 'workspace_name'  

# Instantiate Azure Machine Learning workspace
ws = Workspace.get(name=workspace_name,
                   subscription_id=subscription_id,
                   resource_group=resource_group)
```

### <a name="connect-your-azure-databricks-and-azure-machine-learning-workspaces"></a>Conectar las áreas de trabajo de Azure Databricks y Azure Machine Learning

En [Azure Portal](https://ms.portal.azure.com), puede vincular su área de trabajo de Azure Databricks (ADB) a un área de trabajo de Azure Machine Learning nueva o existente. Para ello, vaya al área de trabajo de ADB y seleccione **Vincular el área de trabajo de Azure Machine Learning**  que está en la parte inferior derecha. Al vincular las áreas de trabajo, podrá realizar un seguimiento de los datos del experimento en el área de trabajo Azure Machine Learning. 

### <a name="mlflow-tracking-in-your-workspaces"></a>Seguimiento de MLflow en las áreas de trabajo

Después de crear una instancia del área de trabajo, el seguimiento de MLflow se establece automáticamente para que se realice el seguimiento en todos los lugares siguientes:

* El área de trabajo vinculada de Azure Machine Learning.
* El área de trabajo ADB original. 
* MLflow. 

Todos los experimentos se redirigirán al servicio de seguimiento administrado de Azure Machine Learning.

#### <a name="set-mlflow-tracking-to-only-track-in-your-azure-machine-learning-workspace"></a>Establecimiento del seguimiento de MLflow para que solo realice el seguimiento en el área de trabajo de Azure Machine Learning

Si prefiere administrar los experimentos con seguimiento en una ubicación centralizada, puede establecer el seguimiento de MLflow para que **solo** realice el seguimiento en el área de trabajo de Azure Machine Learning. 


```python
uri = ws.get_mlflow_tracking_uri()
mlflow.set_tracking_uri(uri)
```

En el script de entrenamiento, importe `mlflow` para usar las API de registro de MLflow y empiece a registrar las métricas de ejecución. En el ejemplo siguiente, se registra la métrica de pérdida de tiempo. 

```python
import mlflow 
mlflow.log_metric('epoch_loss', loss.item()) 
```

## <a name="view-metrics-and-artifacts-in-your-workspace"></a>Visualización de las métricas y los artefactos en el área de trabajo

Las métricas y los artefactos procedentes del registro de MLflow se conservan en el área de trabajo. Para verlos en cualquier momento, vaya al área de trabajo y busque el experimento por su nombre en el área de trabajo en [Azure Machine Learning Studio](https://ml.azure.com).  O bien, ejecute el código siguiente: 

```python
run.get_metrics()
ws.get_details()
```

## <a name="manage-models"></a>Administración de modelos 

Registre y realice un seguimiento de los modelos con el [registro de modelos de Azure Machine Learning](concept-model-management-and-deployment.md#register-package-and-deploy-models-from-anywhere), que admite el registro de modelos de MLflow. Los modelos de Azure Machine Learning se alinean con el esquema de modelos de MLflow, lo que facilita la exportación e importación de estos modelos en diferentes flujos de trabajo. Los metadatos relacionados con MLflow, como el id. de ejecución, también se etiquetan con el modelo registrado para la rastreabilidad. Los usuarios pueden enviar las ejecuciones de entrenamiento, registrar e implementar los modelos generados a partir de las ejecuciones de MLflow. 

Si quiere implementar y registrar el modelo preparado para producción en un solo paso, consulte [Implementación y registro de modelos de MLflow](#deploy-and-register-mlflow-models).

Para registrar y ver un modelo a partir de una ejecución, siga estos pasos:

1. Una vez completada la ejecución, llame al método `register_model()`.

    ```python
    # the model folder produced from the run is registered. This includes the MLmodel file, model.pkl and the conda.yaml.
    run.register_model(model_name = 'my-model', model_path = 'model')
    ```

1. Vea el modelo registrado en su área de trabajo con [Azure Machine Learning Studio](overview-what-is-machine-learning-studio.md)

    En el ejemplo siguiente, el modelo registrado `my-model` tiene etiquetados metadatos de seguimiento de MLflow. 

    ![registered-mlflow-model](./media/how-to-use-mlflow/registered-mlflow-model.png)

1. Seleccione la pestaña **Artefactos** para ver todos los archivos de modelo que se alinean con el esquema de modelos de MLflow (conda.yaml, MLmodel, model.pkl).

    ![model-schema](./media/how-to-use-mlflow/mlflow-model-schema.png)

1. Seleccione MLmodel para ver el archivo MLmodel generado por la ejecución.

    ![MLmodel-schema](./media/how-to-use-mlflow/mlmodel-view.png)



## <a name="deploy-and-register-mlflow-models"></a>Implementación y registro de modelos de MLflow 

La implementación de los experimentos de MLflow como servicio web de Azure Machine Learning le permite aprovechar las funcionalidades de detección del desfase de datos y de administración de modelos de Azure Machine Learning y aplicarlas a los modelos de producción.

Para ello, debe:

1. Registrar el modelo.
1. Determinar qué configuración de implementación quiere usar para su escenario.

    1. [Azure Container Instance (ACI)](#deploy-to-aci) es una opción adecuada para una implementación rápida de desarrollo y pruebas.
    1. [Azure Kubernetes Service (AKS)](#deploy-to-aks) es una opción adecuada para implementaciones de producción escalables.

En el diagrama siguiente se muestra que la API de implementación de MLflow permite implementar los modelos de MLflow existentes como un servicio web de Azure Machine Learning, independientemente de sus marcos de trabajo (PyTorch, TensorFlow, scikit-learn, ONNX, etc.) y administrar los modelos de producción en el área de trabajo.

![ implementación de modelos de mlflow con azure machine learning](./media/how-to-use-mlflow/mlflow-diagram-deploy.png)


### <a name="deploy-to-aci"></a>Implementación en ACI

Establezca la configuración de implementación con el método [deploy_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aciwebservice?view=azure-ml-py#&preserve-view=truedeploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none-). También puede agregar etiquetas y descripciones para ayudar a realizar un seguimiento del servicio web.

```python
from azureml.core.webservice import AciWebservice, Webservice

# Set the model path to the model folder created by your run
model_path = "model"

# Configure 
aci_config = AciWebservice.deploy_configuration(cpu_cores=1, 
                                                memory_gb=1, 
                                                tags={'method' : 'sklearn'}, 
                                                description='Diabetes model',
                                                location='eastus2')
```

A continuación, registre e implemente el modelo en un paso mediante el método [deploy](/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#&preserve-view=truedeploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-) del SDK de Azure Machine Learning. 

```python
(webservice,model) = mlflow.azureml.deploy( model_uri='runs:/{}/{}'.format(run.id, model_path),
                      workspace=ws,
                      model_name='sklearn-model', 
                      service_name='diabetes-model-1', 
                      deployment_config=aci_config, 
                      tags=None, mlflow_home=None, synchronous=True)

webservice.wait_for_deployment(show_output=True)
```

### <a name="deploy-to-aks"></a>Implementación en AKS

Para implementar en AKS, cree primero un clúster de AKS. Cree un clúster de AKS mediante el método [ComputeTarget.create()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.computetarget?view=azure-ml-py#&preserve-view=truecreate-workspace--name--provisioning-configuration-). La creación de un clúster puede tardar entre 20 y 25 minutos.

```python
from azureml.core.compute import AksCompute, ComputeTarget

# Use the default configuration (can also provide parameters to customize)
prov_config = AksCompute.provisioning_configuration()

aks_name = 'aks-mlflow'

# Create the cluster
aks_target = ComputeTarget.create(workspace=ws, 
                                  name=aks_name, 
                                  provisioning_configuration=prov_config)

aks_target.wait_for_completion(show_output = True)

print(aks_target.provisioning_state)
print(aks_target.provisioning_errors)
```
Establezca la configuración de implementación con el método [deploy_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aciwebservice?view=azure-ml-py#&preserve-view=truedeploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none-). También puede agregar etiquetas y descripciones para ayudar a realizar un seguimiento del servicio web.

```python
from azureml.core.webservice import Webservice, AksWebservice

# Set the web service configuration (using default here with app insights)
aks_config = AksWebservice.deploy_configuration(enable_app_insights=True, compute_target_name='aks-mlflow')

```

A continuación, registre e implemente el modelo en un solo paso con el SDK de Azure Machine Learning [deploy()] (a continuación, registre e implemente el modelo con el método [deploy](/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#&preserve-view=truedeploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-) del SDK de Azure Machine Learning. 

```python

# Webservice creation using single command
from azureml.core.webservice import AksWebservice, Webservice

# set the model path 
model_path = "model"

(webservice, model) = mlflow.azureml.deploy( model_uri='runs:/{}/{}'.format(run.id, model_path),
                      workspace=ws,
                      model_name='sklearn-model', 
                      service_name='my-aks', 
                      deployment_config=aks_config, 
                      tags=None, mlflow_home=None, synchronous=True)


webservice.wait_for_deployment()
```

La implementación del servicio puede tardar varios minutos.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no tiene pensado usar los artefactos o las métricas registradas en el área de trabajo, la funcionalidad para eliminarlos de forma individual no está disponible actualmente. Por ello, deberá eliminar el grupo de recursos que contiene la cuenta de almacenamiento y el área de trabajo para no incurrir en cargos:

1. En Azure Portal, seleccione **Grupos de recursos** a la izquierda del todo.

   ![Eliminación en Azure Portal](./media/how-to-use-mlflow/delete-resources.png)

1. En la lista, seleccione el grupo de recursos que creó.

1. Seleccione **Eliminar grupo de recursos**.

1. Escriba el nombre del grupo de recursos. A continuación, seleccione **Eliminar**.

## <a name="example-notebooks"></a>Cuadernos de ejemplo

En [MLflow con cuadernos de Azure ML](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/track-and-monitor-experiments/using-mlflow) se demuestran y se analizan con mayor profundidad los conceptos presentados en este artículo.

## <a name="next-steps"></a>Pasos siguientes

* [Administra sus modelos](concept-model-management-and-deployment.md).
* Supervise los modelos de producción para el [desfase de datos](how-to-monitor-data-drift.md).
