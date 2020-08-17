---
title: Implementación de un modelo para su uso con Cognitive Search
titleSuffix: Azure Machine Learning
description: En este artículo aprenderá a usar Azure Machine Learning para implementar un modelo habilitado para su uso con Azure Cognitive Search. Cognitive Search puede usar modelos implementados por Azure Machine Learning como una aptitud personalizada para enriquecer la experiencia de búsqueda.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: larryfr
author: larryfr
ms.reviewer: larryfr
ms.date: 06/11/2020
ms.openlocfilehash: 59671a0520d665d594356a2e6aee46116a8de5d3
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87541777"
---
# <a name="deploy-a-model-for-use-with-cognitive-search"></a>Implementación de un modelo para su uso con Cognitive Search
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo aprenderá a usar Azure Machine Learning para implementar un modelo para su uso con [Azure Cognitive Search](../search/search-what-is-azure-search.md).

Cognitive Search realiza el procesamiento de contenido sobre un contenido heterogéneo, para que los usuarios o aplicaciones puedan realizar consultas en él. Este proceso se puede mejorar mediante el uso de un modelo implementado a partir de Azure Machine Learning.

Azure Machine Learning puede implementar un modelo entrenado como servicio web. A continuación, el servicio web se inserta en una _aptitud_ de Cognitive Search que se convierte en parte de una canalización de procesamiento.

> [!IMPORTANT]
> La información de este artículo es específica de la implementación del modelo. Proporciona información sobre las configuraciones de implementación admitidas que permiten que Cognitive Search use el modelo.
>
> Para más información sobre cómo configurar Cognitive Search para que use el modelo implementado, consulte el tutorial [Creación e implementación de una aptitud personalizada con Azure Machine Learning](../search/cognitive-search-tutorial-aml-custom-skill.md).
>
> Para obtener el ejemplo en el que se basa el tutorial, consulte [https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill).

Al implementar un modelo para su uso con Azure Cognitive Search, la implementación debe cumplir los siguientes requisitos:

* Use Azure Kubernetes Service para hospedar el modelo para la inferencia.
* Habilite la seguridad de la capa de transporte (TLS) para Azure Kubernetes Service. La seguridad de la capa de transporte se usa para proteger las comunicaciones HTTPS entre Cognitive Search y el modelo implementado.
* El script de entrada debe utilizar el paquete `inference_schema` para generar un esquema de OpenAPI (Swagger) para el servicio.
* El script de entrada también debe aceptar datos JSON como entrada y generarlos como salida.


## <a name="prerequisites"></a>Requisitos previos

* Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

* Un entorno de desarrollo de Python con el SDK de Azure Machine Learning instalado. Para más información, consulte [SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py).  

* Un modelo registrado. Si no tiene ningún modelo, utilice el cuaderno de ejemplo que se encuentra en [https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill).

* Conocimientos generales sobre [cómo y dónde implementar los modelos](how-to-deploy-and-where.md).

## <a name="connect-to-your-workspace"></a>Conexión con su área de trabajo

Un área de trabajo de Azure Machine Learning proporciona un lugar centralizado para trabajar con todos los artefactos que cree al usar Azure Machine Learning. El área de trabajo conserva un historial de las ejecuciones del entrenamiento, que incluye registros, métricas, resultados y una instantánea de sus scripts.

Para conectarse a un área de trabajo existente, use el código siguiente:

> [!IMPORTANT]
> Este fragmento de código espera que la configuración del área de trabajo se guarde en el directorio actual o en su elemento primario. Para más información, consulte [Creación y administración de áreas de trabajo de Azure Machine Learning](how-to-manage-workspace.md). Para más información sobre cómo guardar la configuración en un archivo, consulte [Creación de un archivo de configuración de área de trabajo](how-to-configure-environment.md#workspace).

```python
from azureml.core import Workspace

try:
    # Load the workspace configuration from local cached inffo
    ws = Workspace.from_config()
    print(ws.name, ws.location, ws.resource_group, ws.location, sep='\t')
    print('Library configuration succeeded')
except:
    print('Workspace not found')
```

## <a name="create-a-kubernetes-cluster"></a>Creación de un clúster de Kubernetes

**Tiempo estimado**: aproximadamente 20 minutos.

Un clúster de Kubernetes es un conjunto de instancias de máquinas virtuales (denominadas nodos) que se usan para ejecutar aplicaciones en contenedores.

Al implementar un modelo de Azure Machine Learning en Azure Kubernetes Service, el modelo y todos los recursos necesarios para hospedarlo como un servicio web se empaquetan en un contenedor de Docker. Posteriormente, este contenedor se implementa en el clúster.

En el código siguiente se muestra cómo crear un nuevo clúster de Azure Kubernetes Service para su área de trabajo:

> [!TIP]
> También puede asociar una instancia existente de Azure Kubernetes Service al área de trabajo de Azure Machine Learning. Para más información, consulte [Implementación de modelos en Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md).

> [!IMPORTANT]
> Tenga en cuenta que el código usa el método `enable_ssl()` para habilitar la seguridad de la capa de transporte (TLS) para el clúster. Esto es necesario cuando se planea usar el modelo implementado de Cognitive Services.

```python
from azureml.core.compute import AksCompute, ComputeTarget
# Create or attach to an AKS inferencing cluster

# Create the provisioning configuration with defaults
prov_config = AksCompute.provisioning_configuration()

# Enable TLS (sometimes called SSL) communications
# Leaf domain label generates a name using the formula
#  "<leaf-domain-label>######.<azure-region>.cloudapp.azure.net"
#  where "######" is a random series of characters
prov_config.enable_ssl(leaf_domain_label = "contoso")

cluster_name = 'amlskills'
# Try to use an existing compute target by that name.
# If one doesn't exist, create one.
try:
    
    aks_target = ComputeTarget(ws, cluster_name)
    print("Attaching to existing cluster")
except Exception as e:
    print("Creating new cluster")
    aks_target = ComputeTarget.create(workspace = ws, 
                                  name = cluster_name, 
                                  provisioning_configuration = prov_config)
    # Wait for the create process to complete
    aks_target.wait_for_completion(show_output = True)
```

> [!IMPORTANT]
> Azure le facturará siempre y cuando exista el clúster de AKS. Asegúrese de eliminar el clúster de AKS cuando haya terminado con él.

Para más información acerca del uso de Azure Kubernetes Service con Azure Machine Learning, consulte [Realización de implementaciones en Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md).

## <a name="write-the-entry-script"></a>Escribir el script de entrada

El script de entrada recibe los datos enviados al servicio web, los pasa al modelo y devuelve los resultados de la puntuación. El siguiente script carga el modelo en el inicio y, luego, usa el modelo para puntuar los datos. A veces, este archivo se denomina `score.py`.

> [!TIP]
> El script de entrada es específico para su modelo. Por ejemplo, el script debe conocer el marco que se va a usar con el modelo, los formatos de datos, etc.

> [!IMPORTANT]
> Al planear el uso del modelo implementado desde Azure Cognitive Services, debe usar el paquete `inference_schema` para habilitar la generación de esquemas para la implementación. Este paquete proporciona decoradores que permiten definir el formato de los datos de entrada y salida del servicio web que realiza la inferencia mediante el modelo.

```python
from azureml.core.model import Model
from nlp_architect.models.absa.inference.inference import SentimentInference
from spacy.cli.download import download as spacy_download
import traceback
import json
# Inference schema for schema discovery
from inference_schema.schema_decorators import input_schema, output_schema
from inference_schema.parameter_types.numpy_parameter_type import NumpyParameterType
from inference_schema.parameter_types.standard_py_parameter_type import StandardPythonParameterType

def init():
    """
    Set up the ABSA model for Inference  
    """
    global SentInference
    spacy_download('en')
    aspect_lex = Model.get_model_path('hotel_aspect_lex')
    opinion_lex = Model.get_model_path('hotel_opinion_lex') 
    SentInference = SentimentInference(aspect_lex, opinion_lex)

# Use inference schema decorators and sample input/output to
# build the OpenAPI (Swagger) schema for the deployment
standard_sample_input = {'text': 'a sample input record containing some text' }
standard_sample_output = {"sentiment": {"sentence": "This place makes false booking prices, when you get there, they say they do not have the reservation for that day.", 
                                        "terms": [{"text": "hotels", "type": "AS", "polarity": "POS", "score": 1.0, "start": 300, "len": 6}, 
                                                  {"text": "nice", "type": "OP", "polarity": "POS", "score": 1.0, "start": 295, "len": 4}]}}
@input_schema('raw_data', StandardPythonParameterType(standard_sample_input))
@output_schema(StandardPythonParameterType(standard_sample_output))    
def run(raw_data):
    try:
        # Get the value of the 'text' field from the JSON input and perform inference
        input_txt = raw_data["text"]
        doc = SentInference.run(doc=input_txt)
        if doc is None:
            return None
        sentences = doc._sentences
        result = {"sentence": doc._doc_text}
        terms = []
        for sentence in sentences:
            for event in sentence._events:
                for x in event:
                    term = {"text": x._text, "type":x._type.value, "polarity": x._polarity.value, "score": x._score,"start": x._start,"len": x._len }
                    terms.append(term)
        result["terms"] = terms
        print("Success!")
        # Return the results to the client as a JSON document
        return {"sentiment": result}
    except Exception as e:
        result = str(e)
        # return error message back to the client
        print("Failure!")
        print(traceback.format_exc())
        return json.dumps({"error": result, "tb": traceback.format_exc()})
```

Para obtener más información sobre los scripts de entrada, consulte [Cómo y dónde implementar los modelos](how-to-deploy-and-where.md).

## <a name="define-the-software-environment"></a>Definición del entorno de software

La clase de entorno se usa para definir las dependencias de Python para el servicio. Incluye las dependencias que requiere el modelo y el script de entrada. En este ejemplo, instala los paquetes del índice de PyPI normal, así como de un repositorio de GitHub. 

```python
from azureml.core.conda_dependencies import CondaDependencies 
from azureml.core import Environment

conda = None
pip = ["azureml-defaults", "azureml-monitoring", 
       "git+https://github.com/NervanaSystems/nlp-architect.git@absa", 'nlp-architect', 'inference-schema',
       "spacy==2.0.18"]

conda_deps = CondaDependencies.create(conda_packages=None, pip_packages=pip)

myenv = Environment(name='myenv')
myenv.python.conda_dependencies = conda_deps
```

Para obtener más información sobre los entornos, consulte el tema sobre la [creación y administración de entornos de entrenamiento e implementación](how-to-use-environments.md).

## <a name="define-the-deployment-configuration"></a>Definición de la configuración de la implementación

La configuración de implementación define el entorno de hospedaje de Azure Kubernetes Service que se usa para ejecutar el servicio web.

> [!TIP]
> Si no está seguro de las necesidades de memoria, CPU o GPU de la implementación, puede usar la generación de perfiles para obtener información. Para más información, consulte [Cómo y dónde implementar modelos](how-to-deploy-and-where.md).

```python
from azureml.core.model import Model
from azureml.core.webservice import Webservice
from azureml.core.image import ContainerImage
from azureml.core.webservice import AksWebservice, Webservice

# If deploying to a cluster configured for dev/test, ensure that it was created with enough
# cores and memory to handle this deployment configuration. Note that memory is also used by
# things such as dependencies and AML components.

aks_config = AksWebservice.deploy_configuration(autoscale_enabled=True, 
                                                       autoscale_min_replicas=1, 
                                                       autoscale_max_replicas=3, 
                                                       autoscale_refresh_seconds=10, 
                                                       autoscale_target_utilization=70,
                                                       auth_enabled=True, 
                                                       cpu_cores=1, memory_gb=2, 
                                                       scoring_timeout_ms=5000, 
                                                       replica_max_concurrent_requests=2, 
                                                       max_request_wait_time=5000)
```

Para obtener más información, consulte la documentación de referencia de [AksService.deploy_configuration](/python/api/azureml-core/azureml.core.webservice.akswebservice?view=azure-ml-py#deploy-configuration-autoscale-enabled-none--autoscale-min-replicas-none--autoscale-max-replicas-none--autoscale-refresh-seconds-none--autoscale-target-utilization-none--collect-model-data-none--auth-enabled-none--cpu-cores-none--memory-gb-none--enable-app-insights-none--scoring-timeout-ms-none--replica-max-concurrent-requests-none--max-request-wait-time-none--num-replicas-none--primary-key-none--secondary-key-none--tags-none--properties-none--description-none--gpu-cores-none--period-seconds-none--initial-delay-seconds-none--timeout-seconds-none--success-threshold-none--failure-threshold-none--namespace-none--token-auth-enabled-none--compute-target-name-none-).

## <a name="define-the-inference-configuration"></a>Definición de la configuración de inferencia

La configuración de inferencia apunta al script de entrada y al objeto de entorno:

```python
from azureml.core.model import InferenceConfig
inf_config = InferenceConfig(entry_script='score.py', environment=myenv)
```

Para obtener más información, consulte la documentación de referencia de [InferenceConfig](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.inferenceconfig?view=azure-ml-py).

## <a name="deploy-the-model"></a>Implementación del modelo

Implemente el modelo en el clúster de AKS y espere a que se cree el servicio. En este ejemplo, se cargan dos modelos registrados del registro y se implementan en AKS. Después de la implementación, el archivo `score.py` de la implementación carga estos modelos y los usa para realizar inferencias.

```python
from azureml.core.webservice import AksWebservice, Webservice

c_aspect_lex = Model(ws, 'hotel_aspect_lex')
c_opinion_lex = Model(ws, 'hotel_opinion_lex') 
service_name = "hotel-absa-v2"

aks_service = Model.deploy(workspace=ws,
                           name=service_name,
                           models=[c_aspect_lex, c_opinion_lex],
                           inference_config=inf_config,
                           deployment_config=aks_config,
                           deployment_target=aks_target,
                           overwrite=True)

aks_service.wait_for_deployment(show_output = True)
print(aks_service.state)
```

Para obtener más información, consulte la documentación de referencia del [modelo](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py).

## <a name="issue-a-sample-query-to-your-service"></a>Emisión de una consulta de ejemplo para el servicio

En el ejemplo siguiente se usa la información de implementación que almacenó la sección de código anterior en la variable `aks_service`. Usa esta variable para recuperar la dirección URL de puntuación y el token de autenticación necesarios para comunicarse con el servicio:

```python
import requests
import json

primary, secondary = aks_service.get_keys()

# Test data
input_data = '{"raw_data": {"text": "This is a nice place for a relaxing evening out with friends. The owners seem pretty nice, too. I have been there a few times including last night. Recommend."}}'

# Since authentication was enabled for the deployment, set the authorization header.
headers = {'Content-Type':'application/json',  'Authorization':('Bearer '+ primary)} 

# Send the request and display the results
resp = requests.post(aks_service.scoring_uri, input_data, headers=headers)
print(resp.text)
```

El resultado devuelto desde el servicio es similar al siguiente JSON:

```json
{"sentiment": {"sentence": "This is a nice place for a relaxing evening out with friends. The owners seem pretty nice, too. I have been there a few times including last night. Recommend.", "terms": [{"text": "place", "type": "AS", "polarity": "POS", "score": 1.0, "start": 15, "len": 5}, {"text": "nice", "type": "OP", "polarity": "POS", "score": 1.0, "start": 10, "len": 4}]}}
```

## <a name="connect-to-cognitive-search"></a>Conexión a Cognitive Search

Para más información sobre el uso de este modelo de Cognitive Search, consulte el tutorial [Creación e implementación de una aptitud personalizada con Azure Machine Learning](../search/cognitive-search-tutorial-aml-custom-skill.md).

## <a name="clean-up-the-resources"></a>Limpiar los recursos

Si creó el clúster de AKS específicamente para este ejemplo, elimine los recursos cuando haya terminado de probarlos con Cognitive Search.

> [!IMPORTANT]
> Azure factura en función de cuánto tiempo se implementa el clúster de AKS. Asegúrese de limpiarlo cuando haya terminado con él.

```python
aks_service.delete()
aks_target.delete()
```

## <a name="next-steps"></a>Pasos siguientes

* [Creación e implementación de una aptitud personalizada con Azure Machine Learning](../search/cognitive-search-tutorial-aml-custom-skill.md)
