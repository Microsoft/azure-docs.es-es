---
title: 'Descripción de FPGA: procedimiento para realizar la implementación'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo implementar un servicio web con un modelo que se ejecuta en una FPGA con Azure Machine Learning para obtener una inferencia de latencia ultrabaja.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.author: jordane
author: jpe316
ms.date: 06/03/2020
ms.topic: conceptual
ms.custom: how-to, contperfq4, devx-track-python
ms.openlocfilehash: 7637cc911ea2fbb950a18c2c8d91f5c3eaf02c23
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905080"
---
# <a name="what-are-field-programmable-gate-arrays-fpga-and-how-to-deploy"></a>Descripción de las matrices de puertas programables por campo (FPGA) y procedimiento para realizar la implementación



En este artículo se proporciona una introducción a las matrices de puertas programables por campo (FPGA) y se muestra cómo se pueden implementar los modelos mediante [Azure Machine Learning](overview-what-is-azure-ml.md) en una FPGA de Azure.

## <a name="what-are-fpgas"></a>¿Qué son las FPGA?

Las FPGA contienen una matriz de bloques de lógica programables y una jerarquía de interconexiones reconfigurables. Las interconexiones permiten que estos bloques se configuren de distintas maneras después de su fabricación. Las FPGA ofrecen una combinación de capacidad de programación y rendimiento en comparación con otros chips. 

![Diagrama de comparación de FPGA de Azure Machine Learning](./media/how-to-deploy-fpga-web-service/azure-machine-learning-fpga-comparison.png)

|Procesador| Abreviatura |Descripción|
|---|:-------:|------|
|Circuitos integrados específicos de la aplicación|ASIC|Los circuitos personalizados, como las unidades de procesador TensorFlow (TPU) de Google, proporcionan la máxima eficacia. No pueden reconfigurarse según las necesidades.|
|Matrices de puertas programables por campo|FPGA|Las FPGA como, por ejemplo, las disponibles en Azure, proporcionan un rendimiento similar al de los ASIC. También son flexibles y se pueden volver a configurar con el tiempo, para implementar una lógica nueva.|
|Unidades de procesamiento gráfico|GPU|Una opción popular para los cálculos de inteligencia artificial. GPU ofrece funcionalidades de procesamiento en paralelo, lo que permite una representación de imágenes más rápida que las CPU.|
|Unidades centrales de procesamiento|CPU|Procesadores de uso general cuyo rendimiento no es óptimo para el procesamiento de vídeo y gráficos.|


Las FPGA permiten alcanzar una latencia baja para las solicitudes de inferencia en tiempo real (o puntuación del modelo). Las solicitudes asincrónicas (procesamiento por lotes) no son necesarias. El procesamiento por lotes puede causar latencia porque se deben procesar más datos. Las implementaciones de unidades de procesamiento neuronal no requieren procesamiento por lotes; por lo tanto, la latencia puede ser muchas veces menor en comparación con los procesadores de CPU y GPU.

Puede volver a configurar las FPGA para distintos tipos de modelos de aprendizaje automático. Esta flexibilidad facilita la aceleración de las aplicaciones en función del modelo de memoria y la precisión numérica más óptima que se usa. Como las FPGA se pueden volver a configurar, puede mantenerse al día con los requisitos de los algoritmos de inteligencia artificial que están en constante evolución.

### <a name="fpga-support-in-azure"></a>Compatibilidad con FPGA en Azure

Microsoft Azure es la mayor inversión en la nube del mundo en FPGA. Microsoft usa las FPGA para la evaluación de DNN, la clasificación de búsquedas de Bing y la aceleración de redes definidas por software (SDN) para reducir la latencia y liberar las CPU para otras tareas.

FPGA en Azure se basa en los dispositivos FPGA de Intel, cuyos datos usan los científicos de datos y los desarrolladores para acelerar los cálculos de IA en tiempo real. Esta arquitectura habilitada para FPGA ofrece rendimiento, flexibilidad y escalado, y está disponible en Azure.

Las FPGA de Azure se integran con Azure Machine Learning. Azure puede paralelizar redes neuronales profundas (DNN) entrenadas previamente en las FPGA para realizar el escalado horizontal del servicio. Las DNN pueden entrenarse previamente, como un caracterizador profundo para la transferencia de aprendizaje o ajustarse con cargas actualizadas.

FPGA en Azure es compatible con:

+ Escenarios de reconocimiento y clasificación de imágenes
+ Implementación de TensorFlow (requiere Tensorflow 1.x)
+ Hardware de FPGA de Intel

Actualmente están disponibles los siguientes modelos de DNN:

  - ResNet 50
  - ResNet 152
  - DenseNet-121
  - VGG-16
  - SSD-VGG

  
Las FPGA están disponibles en estas regiones de Azure:
  - Este de EE. UU.
  - Sudeste de Asia
  - Oeste de Europa
  - Oeste de EE. UU. 2

Para optimizar el rendimiento y la latencia, el envío de datos del cliente al modelo de FPGA debe hacerse en una de las regiones anteriores (aquella en la que se haya implementado el modelo).

La **familia PBS de máquinas virtuales de Azure** contiene matrices FPGA Intel Arria 10. Esto se mostrará como "Standard PBS Family vCPUs" (vCPU de la familia PBS estándar) al revisar la asignación de cuota de Azure. La máquina virtual PB6 tiene seis vCPU y una FPGA, y Azure ML la aprovisionará automáticamente como parte de la implementación de un modelo en una FPGA. Solo se usa con Azure ML y no puede ejecutar secuencias de bits arbitrarias. Por ejemplo, no podrá incorporar a la FPGA secuencias de bits para realizar cifrado, codificación, etcétera.


## <a name="deploy-models-on-fpgas"></a>Implementación de modelos en FPGA

Puede implementar un modelo como servicio web en FPGA con [modelos acelerados mediante hardware de Azure Machine Learning](https://docs.microsoft.com/python/api/azureml-accel-models/azureml.accel?view=azure-ml-py&preserve-view=true). El uso de las FPGA brinda una inferencia de latencia ultrabaja, incluso con un tamaño de lote único. Inferencia o modelo de puntuación, es la fase donde se usa el modelo implementado para la predicción, frecuentemente en datos de producción.

La implementación de un modelo en una FPGA conlleva los siguientes pasos:

1. Definir el modelo de TensorFlow
1. Convertir el modelo a ONNX
1. Implementar el modelo en la nube o en un dispositivo perimetral
1. Consumo del modelo implementado

En este ejemplo, se crea un gráfico de TensorFlow para preprocesar la imagen de entrada, se caracteriza mediante ResNet 50 en una FPGA y, a continuación, se ejecutan las características mediante un clasificador entrenado en el conjunto de datos de ImageNet. A continuación, el modelo se implementa en un clúster de AKS.

### <a name="prerequisites"></a>Prerrequisitos

- Suscripción a Azure. Si no tiene una, tendrá que crear una cuenta de [pago por uso](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go) (las cuentas de Azure gratuitas no son válidas para la cuota de FPGA).
- [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)
- Cuota de FPGA. Use la CLI de Azure para comprobar si dispone de cuota:

    ```azurecli-interactive
    az vm list-usage --location "eastus" -o table --query "[?localName=='Standard PBS Family vCPUs']"
    ```


    El comando devuelve un texto similar al siguiente:

    ```text
    CurrentValue    Limit    LocalName
    --------------  -------  -------------------------
    0               6        Standard PBS Family vCPUs
    ```

    Asegúrese de que tiene al menos 6 vCPU en __CurrentValue__.

    Si no tiene cuota, envíe una solicitud a [https://aka.ms/accelerateAI](https://aka.ms/accelerateAI).

- Un área de trabajo de Azure Machine Learning y el SDK de Azure Machine Learning para Python instalado. Para obtener más información, consulte [Create a workspace](how-to-manage-workspace.md) (Crear un área de trabajo).
 
- El SDK de Python para modelos acelerados por hardware:

    ```bash
    pip install --upgrade azureml-accel-models[cpu]
    ```
### <a name="1-define-the-tensorflow-model"></a>1. Definir el modelo de TensorFlow

Use el [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py&preserve-view=true) para crear una definición de servicio. Una definición de servicio es un archivo que describe una canalización de grafos (entrada, caracterizador y clasificador) basado en TensorFlow. El comando de implementación comprime automáticamente la definición y los grafos en un archivo ZIP, que cargará en Azure Blog Storage. La DNN ya está implementada para ejecutarse en la FPGA.

1. Un área de trabajo de Azure Machine Learning.

   ```python
   import os
   import tensorflow as tf
   
   from azureml.core import Workspace
  
   ws = Workspace.from_config()
   print(ws.name, ws.resource_group, ws.location, ws.subscription_id, sep='\n')
   ```

1. Preprocese la imagen. La entrada al servicio web es una imagen JPEG.  El primer paso es descodificar la imagen JPEG y preprocesarla.  Las imágenes JPEG se tratan como cadenas y el resultado son tensores que serán la entrada del modelo ResNet 50.

   ```python
   # Input images as a two-dimensional tensor containing an arbitrary number of images represented a strings
   import azureml.accel.models.utils as utils
   tf.reset_default_graph()
   
   in_images = tf.placeholder(tf.string)
   image_tensors = utils.preprocess_array(in_images)
   print(image_tensors.shape)
   ```

1. Cargue un caracterizador. Inicialice el modelo y descargue un punto de control de TensorFlow de la versión cuantificada de ResNet50 que se usará como caracterizador.  Puede reemplazar "QuantizedResnet50" en el fragmento de código siguiente mediante la importación de otras redes neuronales profundas:

   - QuantizedResnet152
   - QuantizedVgg16
   - Densenet121

   ```python
   from azureml.accel.models import QuantizedResnet50
   save_path = os.path.expanduser('~/models')
   model_graph = QuantizedResnet50(save_path, is_frozen=True)
   feature_tensor = model_graph.import_graph_def(image_tensors)
   print(model_graph.version)
   print(feature_tensor.name)
   print(feature_tensor.shape)
   ```

1. Agregue un clasificador. Este clasificador se ha entrenado en el conjunto de datos de ImageNet.  Hay ejemplos disponibles para transferir aprendizaje y entrenamiento de las ponderaciones personalizadas en el conjunto de [cuadernos de ejemplo](https://github.com/Azure/MachineLearningNotebooks).

   ```python
   classifier_output = model_graph.get_default_classifier(feature_tensor)
   print(classifier_output)
   ```

1. Guarde el modelo. Ahora que se han cargado el clasificador, el preprocesador y caracterizador de ResNet 50, guarde el gráfico y las variables asociadas como modelo.

   ```python
   model_name = "resnet50"
   model_save_path = os.path.join(save_path, model_name)
   print("Saving model in {}".format(model_save_path))
   
   with tf.Session() as sess:
       model_graph.restore_weights(sess)
       tf.saved_model.simple_save(sess, model_save_path,
                                  inputs={'images': in_images},
                                  outputs={'output_alias': classifier_output})
   ```

1. Guarde los tensores de entrada y salida. Los tensores de entrada y salida que se han creado durante los pasos de preprocesamiento y el clasificador serán necesarios para la inferencia y la conversión del modelo.

   ```python
   input_tensors = in_images.name
   output_tensors = classifier_output.name

   print(input_tensors)
   print(output_tensors)
   ```

   > [!IMPORTANT]
   > Guarde los tensores de entrada y salida porque los necesitará para las solicitudes de inferencia y conversión del modelo.

   Los modelos disponibles y los tensores de salida del clasificador predeterminado correspondiente se muestran a continuación, y es lo que se usaría para la inferencia si se ha usado el clasificador predeterminado.

   + Resnet50, QuantizedResnet50
     ```python
     output_tensors = "classifier_1/resnet_v1_50/predictions/Softmax:0"
     ```
   + Resnet152, QuantizedResnet152
     ```python
     output_tensors = "classifier/resnet_v1_152/predictions/Softmax:0"
     ```
   + Densenet121, QuantizedDensenet121
     ```python
     output_tensors = "classifier/densenet121/predictions/Softmax:0"
     ```
   + Vgg16, QuantizedVgg16
     ```python
     output_tensors = "classifier/vgg_16/fc8/squeezed:0"
     ```
   + SsdVgg, QuantizedSsdVgg
     ```python
     output_tensors = ['ssd_300_vgg/block4_box/Reshape_1:0', 'ssd_300_vgg/block7_box/Reshape_1:0', 'ssd_300_vgg/block8_box/Reshape_1:0', 'ssd_300_vgg/block9_box/Reshape_1:0', 'ssd_300_vgg/block10_box/Reshape_1:0', 'ssd_300_vgg/block11_box/Reshape_1:0', 'ssd_300_vgg/block4_box/Reshape:0', 'ssd_300_vgg/block7_box/Reshape:0', 'ssd_300_vgg/block8_box/Reshape:0', 'ssd_300_vgg/block9_box/Reshape:0', 'ssd_300_vgg/block10_box/Reshape:0', 'ssd_300_vgg/block11_box/Reshape:0']
     ```

### <a name="2-convert-the-model"></a>2. Convertir el modelo

Antes de implementar el modelo en las FPGA, tiene que convertirlo al formato ONNX.

1. [Registre](concept-model-management-and-deployment.md) el modelo mediante el SDK con el archivo .zip en Azure Blob Storage. Agregar etiquetas y otros metadatos sobre el modelo le ayuda a realizar el seguimiento de los modelos entrenados.

   ```python
   from azureml.core.model import Model

   registered_model = Model.register(workspace=ws,
                                     model_path=model_save_path,
                                     model_name=model_name)

   print("Successfully registered: ", registered_model.name,
         registered_model.description, registered_model.version, sep='\t')
   ```

   Si ya se ha registrado un modelo y quiere cargarlo, puede recuperarlo.

   ```python
   from azureml.core.model import Model
   model_name = "resnet50"
   # By default, the latest version is retrieved. You can specify the version, i.e. version=1
   registered_model = Model(ws, name="resnet50")
   print(registered_model.name, registered_model.description,
         registered_model.version, sep='\t')
   ```

1. Convierta el gráfico de TensorFlow al formato de intercambio de red neuronal abierta ([ONNX](https://onnx.ai/)).  Deberá proporcionar los nombres de los tensores de entrada y salida y estos nombres los utilizará el cliente cuando se consuma el servicio web.

   ```python
   from azureml.accel import AccelOnnxConverter

   convert_request = AccelOnnxConverter.convert_tf_model(
       ws, registered_model, input_tensors, output_tensors)

   # If it fails, you can run wait_for_completion again with show_output=True.
   convert_request.wait_for_completion(show_output=False)

   # If the above call succeeded, get the converted model
   converted_model = convert_request.result
   print("\nSuccessfully converted: ", converted_model.name, converted_model.url, converted_model.version,
         converted_model.id, converted_model.created_time, '\n')
   ```

### <a name="3-containerize-and-deploy-the-model"></a>3. Incluir en contenedor e implementar el modelo

Cree la imagen de Docker a partir del modelo convertido y todas las dependencias.  A continuación, se puede implementar esta imagen de Docker y se puede crear una instancia de ella.  Entre los destinos de implementación admitidos se incluye AKS en la nube o un dispositivo perimetral como [Azure Data Box Edge](https://docs.microsoft.com/azure/databox-online/data-box-edge-overview).  También puede agregar etiquetas y descripciones para la imagen de Docker registrada.

   ```python
   from azureml.core.image import Image
   from azureml.accel import AccelContainerImage

   image_config = AccelContainerImage.image_configuration()
   # Image name must be lowercase
   image_name = "{}-image".format(model_name)

   image = Image.create(name=image_name,
                        models=[converted_model],
                        image_config=image_config,
                        workspace=ws)
   image.wait_for_creation(show_output=False)
   ```

   Enumere las imágenes por etiqueta y obtenga los registros detallados para cualquier tipo de depuración.

   ```python
   for i in Image.list(workspace=ws):
       print('{}(v.{} [{}]) stored at {} with build log {}'.format(
           i.name, i.version, i.creation_state, i.image_location, i.image_build_log_uri))
   ```

#### <a name="deploy-to-aks-cluster"></a>Implementación en un clúster de AKS

1. Para implementar el modelo como un servicio web de producción a gran escala, use Azure Kubernetes Service (AKS). Puede crear uno con el SDK de Azure Machine Learning, la CLI o [Azure Machine Learning Studio](https://ml.azure.com).

    ```python
    from azureml.core.compute import AksCompute, ComputeTarget
    
    # Specify the Standard_PB6s Azure VM and location. Values for location may be "eastus", "southeastasia", "westeurope", or "westus2". If no value is specified, the default is "eastus".
    prov_config = AksCompute.provisioning_configuration(vm_size = "Standard_PB6s",
                                                        agent_count = 1,
                                                        location = "eastus")
    
    aks_name = 'my-aks-cluster'
    # Create the cluster
    aks_target = ComputeTarget.create(workspace=ws,
                                      name=aks_name,
                                      provisioning_configuration=prov_config)
    ```

    La implementación de AKS puede tardar unos 15 minutos.  Compruebe que la implementación se ha realizado correctamente.

    ```python
    aks_target.wait_for_completion(show_output=True)
    print(aks_target.provisioning_state)
    print(aks_target.provisioning_errors)
    ```

1. Implemente el contenedor en el clúster de AKS.

    ```python
    from azureml.core.webservice import Webservice, AksWebservice
    
    # For this deployment, set the web service configuration without enabling auto-scaling or authentication for testing
    aks_config = AksWebservice.deploy_configuration(autoscale_enabled=False,
                                                    num_replicas=1,
                                                    auth_enabled=False)
    
    aks_service_name = 'my-aks-service'
    
    aks_service = Webservice.deploy_from_image(workspace=ws,
                                               name=aks_service_name,
                                               image=image,
                                               deployment_config=aks_config,
                                               deployment_target=aks_target)
    aks_service.wait_for_deployment(show_output=True)
    ```

#### <a name="deploy-to-a-local-edge-server"></a>Implementación en un servidor perimetral local

Todos los [dispositivos perimetrales de Azure Data Box Edge](https://docs.microsoft.com/azure/databox-online/data-box-edge-overview
) contienen una FPGA para ejecutar el modelo.  Solo se puede ejecutar un único modelo en la FPGA al mismo tiempo.  Para ejecutar otro modelo, basta con implementar un nuevo contenedor. Pueden encontrarse instrucciones y código de ejemplo en [este ejemplo de Azure](https://github.com/Azure-Samples/aml-hardware-accelerated-models).

### <a name="4-consume-the-deployed-model"></a>4. Consumo del modelo implementado

La imagen de Docker admite gRPC y la API de "predicción" de TensorFlow Serving.  Use el cliente de ejemplo para llamar a la imagen de Docker y obtener predicciones a partir del modelo.  Hay código de cliente de ejemplo disponible:
- [Python](https://github.com/Azure/aml-real-time-ai/blob/master/pythonlib/amlrealtimeai/client.py)
- [C#](https://github.com/Azure/aml-real-time-ai/blob/master/sample-clients/csharp)

Si quiere usar TensorFlow Serving, puede [descargar un cliente de ejemplo](https://www.tensorflow.org/serving/setup).

```python
# Using the grpc client in Azure ML Accelerated Models SDK package
from azureml.accel import PredictionClient

address = aks_service.scoring_uri
ssl_enabled = address.startswith("https")
address = address[address.find('/')+2:].strip('/')
port = 443 if ssl_enabled else 80

# Initialize Azure ML Accelerated Models client
client = PredictionClient(address=address,
                          port=port,
                          use_ssl=ssl_enabled,
                          service_name=aks_service.name)
```

Puesto que este clasificador se ha entrenado en el conjunto de datos [ImageNet](http://www.image-net.org/), asigne las clases a etiquetas de lenguaje natural.

```python
import requests
classes_entries = requests.get(
    "https://raw.githubusercontent.com/Lasagne/Recipes/master/examples/resnet50/imagenet_classes.txt").text.splitlines()

# Score image with input and output tensor names
results = client.score_file(path="./snowleopardgaze.jpg",
                            input_name=input_tensors,
                            outputs=output_tensors)

# map results [class_id] => [confidence]
results = enumerate(results)
# sort results by confidence
sorted_results = sorted(results, key=lambda x: x[1], reverse=True)
# print top 5 results
for top in sorted_results[:5]:
    print(classes_entries[top[0]], 'confidence:', top[1])
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Elimine el servicio web, la imagen y el modelo (se debe realizar en este orden, ya que existen dependencias).

```python
aks_service.delete()
aks_target.delete()
image.delete()
registered_model.delete()
converted_model.delete()
```

## <a name="next-steps"></a>Pasos siguientes

Consulte estos cuadernos, vídeos y blogs:

+ Varios [cuadernos de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/deployment/accelerated-models).
+ Para proteger los servicios web FPGA, consulte el documento [Protección de servicios web](how-to-secure-web-service.md).
+ [Hardware de hiperescala: ML at scale on top of Azure + FPGA: Build 2018 (video)](https://channel9.msdn.com/events/Build/2018/BRK3202) (Hardware de hiperescala: ML a escala sobre Azure + FPGA: compilación 2018 [vídeo])
+ [Inside the Microsoft FPGA-based configurable cloud (video)](https://channel9.msdn.com/Events/Build/2017/B8063) (Dentro de la nube configurable basada en FPGA de Microsoft [vídeo])
+ [Project Brainwave for real-time AI](https://www.microsoft.com/research/project/project-brainwave/) (Project Brainwave para IA en tiempo real): página principal del proyecto
+ [Sistema automatizado de inspección óptica](https://blogs.microsoft.com/ai/build-2018-project-brainwave/)
+ [Asignación de uso de terrenos](https://blogs.technet.microsoft.com/machinelearning/2018/05/29/how-to-use-fpgas-for-deep-learning-inference-to-perform-land-cover-mapping-on-terabytes-of-aerial-images/)
