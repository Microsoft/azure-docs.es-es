---
title: Seguimiento de MLflow para experimentos de Machine Learning
titleSuffix: Azure Machine Learning
description: Configure MLflow con Azure Machine Learning para registrar métricas y artefactos de modelos de aprendizaje automático e implementar los modelos de aprendizaje automático como un servicio web.
services: machine-learning
author: rastala
ms.author: roastala
ms.service: machine-learning
ms.subservice: core
ms.reviewer: nibaccam
ms.date: 06/04/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: a0241864a5eafe8783aea463197f86ff949ea9ed
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87853388"
---
# <a name="track-model-metrics-and-deploy-ml-models-with-mlflow-and-azure-machine-learning-preview"></a>Seguimiento de métricas del modelo e implementación de modelos de aprendizaje automático con MLflow y Azure Machine Learning (versión preliminar)

[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo se muestra cómo habilitar el identificador URI de seguimiento y la API de registro de MLflow, que en conjunto se conocen como [MLflow Tracking](https://mlflow.org/docs/latest/quickstart.html#using-the-tracking-api), para conectar experimentos de MLflow y Azure Machine Learning.  Esto le permite:

+ Realizar un seguimiento de las métricas y los artefactos de los experimentos, así como registrarlos, en el [área de trabajo de Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/concept-azure-machine-learning-architecture#workspaces). Si ya usa Seguimiento de MLflow para los experimentos, el área de trabajo proporciona una ubicación centralizada, segura y escalable para almacenar los modelos y las métricas de entrenamiento.

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

 El Seguimiento de MLflow ofrece funciones de registro de métricas y almacenamiento de artefactos que de otra manera solo están disponibles a través del [SDK de Python de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py).


| Capacidad | Implementación y seguimiento de&nbsp;MLflow | SDK de Python de Azure Machine Learning |  CLI de Azure Machine Learning | Azure Machine Learning Studio|
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
* [Instale el SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py) en el equipo local. El SDK proporciona la conectividad de MLflow para acceder al área de trabajo.
* [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

## <a name="track-local-runs"></a>Seguimiento de ejecuciones locales

El seguimiento de MLflow con Azure Machine Learning le permite almacenar las métricas y los artefactos registrados desde las ejecuciones locales en el área de trabajo de Azure Machine Learning.

Instale el paquete `azureml-mlflow` que va a usar el Seguimiento de MLflow con Azure Machine Learning en los experimentos ejecutados localmente en un editor de código o Jupyter Notebook.

```shell
pip install azureml-mlflow
```

Importe las clases `mlflow` y [`Workspace`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace(class)?view=azure-ml-py) para acceder al URI de seguimiento de MLflow y configurar el área de trabajo.

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

Las ejecuciones remotas permiten entrenar modelos en procesos más eficaces, como máquinas virtuales habilitadas para GPU o clústeres de Proceso de Machine Learning. Consulte [Configuración de destinos de proceso del entrenamiento del modelo](how-to-set-up-training-targets.md) para obtener información sobre las distintas opciones de proceso.

Configure el proceso y el entorno de ejecución de entrenamiento con la clase [`Environment`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment.environment?view=azure-ml-py). Incluya paquetes de PIP `mlflow` y `azureml-mlflow` en la sección [`CondaDependencies`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.conda_dependencies.condadependencies?view=azure-ml-py) del entorno. Después, construya [`ScriptRunConfig`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.script_run_config.scriptrunconfig?view=azure-ml-py) con el proceso remoto como destino de proceso.

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

Con esta configuración del proceso y de la ejecución de entrenamiento, use el método `Experiment.submit('train.py')` para enviar una ejecución. Este método establece automáticamente el identificador URI de seguimiento de MLflow y dirige el registro de MLflow al área de trabajo.

```Python
run = exp.submit(src)
```

## <a name="track-azure-databricks-runs"></a>Seguimiento de las ejecuciones de Azure Databricks

MLflow Tracking con Azure Machine Learning permite almacenar las métricas y los artefactos de las ejecuciones de Azure Databricks en el área de trabajo de Azure Machine Learning.

Para ejecutar los experimentos de Mlflow con Azure Databricks, primero debe crear un [área de trabajo y un clúster de Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/quickstart-create-databricks-workspace-portal). En su clúster, compruebe que instala la biblioteca *azureml-mlflow* desde PyPi para asegurarse de que el clúster tiene acceso a las funciones y clases necesarias.

Una vez hecho esto, importe su cuaderno de experimentos, adjúntelo a su clúster de Azure Databricks y ejecute el experimento. 

### <a name="install-libraries"></a>Instalar bibliotecas

Para instalar bibliotecas en el clúster, vaya a la pestaña **Bibliotecas** y haga clic en **Instalar nuevo**.

 ![Diagrama de MLflow con Azure Machine Learning](./media/how-to-use-mlflow/azure-databricks-cluster-libraries.png)

En el campo **Paquete**, escriba azureml-mlflow y, a continuación, haga clic en instalar. Repita este paso según sea necesario para instalar otros paquetes adicionales en el clúster para el experimento.

 ![Diagrama de MLflow con Azure Machine Learning](./media/how-to-use-mlflow/install-libraries.png)

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

#### <a name="connect-your-azure-databricks-and-azure-machine-learning-workspaces"></a>Conectar las áreas de trabajo de Azure Databricks y Azure Machine Learning

En [Azure Portal](https://ms.portal.azure.com), puede vincular su área de trabajo de Azure Databricks (ADB) a un área de trabajo de Azure Machine Learning nueva o existente. Para ello, vaya al área de trabajo de ADB y seleccione **Vincular el área de trabajo de Azure Machine Learning**  que está en la parte inferior derecha. Al vincular las áreas de trabajo, podrá realizar un seguimiento de los datos del experimento en el área de trabajo Azure Machine Learning. 

### <a name="link-mlflow-tracking-to-your-workspace"></a>Vinculación del seguimiento de MLflow al área de trabajo

Una vez cree la instancia de su área de trabajo, establezca el URI de seguimiento de MLflow. Al hacerlo, vincula el seguimiento de MLflow al área de trabajo de Azure Machine Learning. Después de la vinculación, todos los experimentos se redirigirán al servicio de seguimiento administrado de Azure Machine Learning.

#### <a name="directly-set-mlflow-tracking-in-your-notebook"></a>Establecer directamente el seguimiento de MLflow en el cuaderno

```python
uri = ws.get_mlflow_tracking_uri()
mlflow.set_tracking_uri(uri)
```

En su script de entrenamiento, importe mlflow para usar las API de registro de MLflow y empiece a registrar las métricas de ejecución. En el ejemplo siguiente, se registra la métrica de pérdida de tiempo. 

```python
import mlflow 
mlflow.log_metric('epoch_loss', loss.item()) 
```

#### <a name="automate-setting-mlflow-tracking"></a>Automatizar el seguimiento de la configuración de MLflow

En lugar de configurar manualmente el URI de seguimiento en cada sesión del cuaderno del experimento posterior en los clústeres, hágalo automáticamente mediante este [script de inicio del clúster de seguimiento de Azure Machine Learning ](https://github.com/Azure/MachineLearningNotebooks/blob/3ce779063b000e0670bdd1acc6bc3a4ee707ec13/how-to-use-azureml/azure-databricks/linking/README.md).

Cuando se configura correctamente, puede ver los datos de seguimiento de MLflow en la API de REST de Azure Machine Learning, en todos los clientes y en Azure Databricks a través de la interfaz de usuario de MLflow o mediante el cliente de MLflow.

## <a name="view-metrics-and-artifacts-in-your-workspace"></a>Visualización de las métricas y los artefactos en el área de trabajo

Las métricas y los artefactos procedentes del registro de MLflow se conservan en el área de trabajo. Para verlos en cualquier momento, vaya al área de trabajo y busque el experimento por su nombre en el área de trabajo en [Azure Machine Learning Studio](https://ml.azure.com).  O bien, ejecute el código siguiente: 

```python
run.get_metrics()
ws.get_details()
```

## <a name="deploy-mlflow-models-as-a-web-service"></a>Implementación de modelos de MLflow como servicio web

La implementación de los experimentos de MLflow como servicio web de Azure Machine Learning le permite aprovechar las funcionalidades de detección del desfase de datos y de administración de modelos de Azure Machine Learning y aplicarlas a los modelos de producción.

En el diagrama siguiente se muestra que la API de implementación de MLflow permite implementar los modelos de MLflow existentes como un servicio web de Azure Machine Learning, independientemente de sus marcos de trabajo (PyTorch, TensorFlow, scikit-learn, ONNX, etc.) y administrar los modelos de producción en el área de trabajo.

![Diagrama de MLflow con Azure Machine Learning](./media/how-to-use-mlflow/mlflow-diagram-deploy.png)

### <a name="log-your-model"></a>Registro del modelo

Antes de implementarlo, asegúrese de que el modelo está guardado a fin de poder hacer referencia a él y a la ubicación de su ruta de acceso en la implementación. En el script de entrenamiento, debe haber un código similar al siguiente método [mlflow.sklearn.log_model()](https://www.mlflow.org/docs/latest/python_api/mlflow.sklearn.html) , que guarda su modelo en el directorio de salidas especificado. 

```python
# change sklearn to pytorch, tensorflow, etc. based on your experiment's framework 
import mlflow.sklearn

# Save the model to the outputs directory for capture
mlflow.sklearn.log_model(regression_model, model_save_path)
```
>[!NOTE]
> Incluya el parámetro `conda_env` para pasar una representación del diccionario de las dependencias y el entorno en los que se debe ejecutar este modelo.

### <a name="retrieve-model-from-previous-run"></a>Recuperación del modelo de la ejecución anterior

Para recuperar la ejecución, necesita su identificador y la ruta de acceso del historial de ejecución de la ubicación en que se guardó el modelo. 

```python
# gets the list of runs for your experiment as an array
experiment_name = 'experiment-with-mlflow'
exp = ws.experiments[experiment_name]
runs = list(exp.get_runs())

# get the run ID and the path in run history
runid = runs[0].id
model_save_path = 'model'
```

### <a name="deploy-the-model"></a>Implementación del modelo

Use el SDK de Azure Machine Learning para implementar el modelo como servicio web.

Primero, especifique la configuración de la implementación. Azure Container Instance (ACI) es una opción adecuada para una implementación rápida de desarrollo y pruebas, mientras que Azure Kubernetes Service (AKS) resulta apropiado para implementaciones de producción escalables.

#### <a name="deploy-to-aci"></a>Implementación en ACI

Establezca la configuración de implementación con el método [deploy_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aciwebservice?view=azure-ml-py#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none-). También puede agregar etiquetas y descripciones para ayudar a realizar un seguimiento del servicio web.

```python
from azureml.core.webservice import AciWebservice, Webservice

# Configure 
aci_config = AciWebservice.deploy_configuration(cpu_cores=1, 
                                                memory_gb=1, 
                                                tags={'method' : 'sklearn'}, 
                                                description='Diabetes model',
                                                location='eastus2')
```

A continuación, registre e implemente el modelo mediante el método [deploy](/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#deploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-) del SDK de Azure Machine Learning. 

```python
(webservice,model) = mlflow.azureml.deploy( model_uri='runs:/{}/{}'.format(run.id, model_path),
                      workspace=ws,
                      model_name='sklearn-model', 
                      service_name='diabetes-model-1', 
                      deployment_config=aci_config, 
                      tags=None, mlflow_home=None, synchronous=True)

webservice.wait_for_deployment(show_output=True)
```
#### <a name="deploy-to-aks"></a>Implementación en AKS

Para implementar en AKS, cree primero un clúster de AKS. Cree un clúster de AKS mediante el método [ComputeTarget.create()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.computetarget?view=azure-ml-py#create-workspace--name--provisioning-configuration-). La creación de un clúster puede tardar entre 20 y 25 minutos.

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
Establezca la configuración de implementación con el método [deploy_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aciwebservice?view=azure-ml-py#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none-). También puede agregar etiquetas y descripciones para ayudar a realizar un seguimiento del servicio web.

```python
from azureml.core.webservice import Webservice, AksWebservice

# Set the web service configuration (using default here with app insights)
aks_config = AksWebservice.deploy_configuration(enable_app_insights=True, compute_target_name='aks-mlflow')


Then, deploy the image by using the Azure Machine Learning SDK [deploy()](Then, register and deploy the model by using the Azure Machine Learning SDK [deploy](/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#deploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-) method. 

```python
# Webservice creation using single command
from azureml.core.webservice import AksWebservice, Webservice
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

En [MLflow con cuadernos de Azure ML](https://aka.ms/azureml-mlflow-examples) se demuestran y se analizan con mayor profundidad los conceptos presentados en este artículo.

## <a name="next-steps"></a>Pasos siguientes
* [Administra sus modelos](concept-model-management-and-deployment.md).
* Supervise los modelos de producción para el [desfase de datos](how-to-monitor-data-drift.md).
