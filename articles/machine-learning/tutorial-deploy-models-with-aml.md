---
title: 'Tutorial de clasificación de imágenes: Implementación de modelos'
titleSuffix: Azure Machine Learning
description: En este tutorial, que es el segundo de una serie de dos partes, se muestra cómo usar Azure Machine Learning para implementar un modelo de clasificación de imágenes con scikit-learn en un cuaderno de Jupyter en Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: sdgilley
ms.author: sgilley
ms.date: 03/18/2020
ms.custom: seodec18
ms.openlocfilehash: f0d78fc55ca60fa883c742885acf2fa98ede61ad
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90896583"
---
# <a name="tutorial-deploy-an-image-classification-model-in-azure-container-instances"></a>Tutorial: Implementación de un modelo de clasificación de imágenes en Azure Container Instances


Este tutorial es la **segunda parte de dos**. En el [tutorial anterior](tutorial-train-models-with-aml.md), entrenó modelos de aprendizaje automático y registró un modelo en su área de trabajo en la nube.  Ya está listo para implementar el modelo como un servicio web. Un servicio web es una imagen, en este caso, una imagen de Docker. Encapsula la lógica de puntuación y el propio modelo. 

En esta parte del tutorial, se usa Azure Machine Learning para las siguientes tareas:

> [!div class="checklist"]
> * Configuración del entorno de pruebas.
> * Recuperación del modelo del área de trabajo.
> * Implementación del modelo en Container Instances.
> * Prueba del modelo implementado.

Container Instances es una excelente solución para probar y conocer el flujo de trabajo. Para implementaciones de producción escalables, considere la posibilidad de usar Azure Kubernetes Service. Para más información, consulte [cómo y dónde realizar la implementación](how-to-deploy-and-where.md).

>[!NOTE]
> El código de este artículo se probó con la versión 1.0.83 del SDK de Azure Machine Learning.

## <a name="prerequisites"></a>Prerrequisitos

Para ejecutar el cuaderno, complete primero el entrenamiento del modelo en [Tutorial (parte 1): Entrenamiento de un modelo de clasificación de imágenes](tutorial-train-models-with-aml.md).   A continuación, abra el cuaderno *img-classification-part2-deploy.ipynb* clonado de la carpeta *tutorials/image-classification-mnist-data* clonada.

Este tutorial también está disponible en [GitHub](https://github.com/Azure/MachineLearningNotebooks/tree/master/tutorials) si quiere usarlo en su propio [entorno local](how-to-configure-environment.md#local).  Asegúrese de que ha instalado `matplotlib` y `scikit-learn` en su entorno. 

> [!Important]
> El resto de este artículo contiene el mismo contenido que se ve en el cuaderno.  
>
> Cambie ahora al cuaderno de Jupyter Notebook si desea leer a medida que ejecuta el código.
> Para ejecutar una sola celda de código en un cuaderno, haga clic en la celda y presione **Mayús + Entrar**. O bien, ejecute el cuaderno completo, para lo que debe elegir **Ejecutar todo** en la barra de herramientas superior.

## <a name="set-up-the-environment"></a><a name="start"></a>Configuración del entorno

Empiece por configurar un entorno de prueba.

### <a name="import-packages"></a>Importación de paquetes

Importe los paquetes de Python necesarios para este tutorial.


```python
%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt
 
import azureml.core

# Display the core SDK version number
print("Azure ML SDK Version: ", azureml.core.VERSION)
```

## <a name="deploy-as-web-service"></a>Implementación como servicio web

Implementación del modelo como un servicio web hospedado en ACI 

Para crear el entorno correcto para ACI, proporcione lo siguiente:
* Un script de puntuación para mostrar cómo usar el modelo.
* Un archivo de configuración para compilar el ACI.
* El modelo ya entrenado.

### <a name="create-scoring-script"></a>Creación del script de puntuación

Cree el script de puntuación, llamado score.py, que usó la llamada al servicio web para mostrar cómo usar el modelo.

Debe incluir dos funciones necesarias en el script de puntuación:
* La función `init()`, que normalmente carga el modelo en un objeto global. Esta función solo se ejecuta una vez cuando se inicia el contenedor de Docker. 

* La función `run(input_data)` usa el modelo para predecir un valor basado en los datos de entrada. Los datos de entrada y salida de la ejecución suelen usan el formato JSON para la serialización y deserialización, pero se admiten otros formatos.

```python
%%writefile score.py
import json
import numpy as np
import os
import pickle
import joblib

def init():
    global model
    # AZUREML_MODEL_DIR is an environment variable created during deployment.
    # It is the path to the model folder (./azureml-models/$MODEL_NAME/$VERSION)
    # For multiple models, it points to the folder containing all deployed models (./azureml-models)
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')
    model = joblib.load(model_path)

def run(raw_data):
    data = np.array(json.loads(raw_data)['data'])
    # make prediction
    y_hat = model.predict(data)
    # you can return any data type as long as it is JSON-serializable
    return y_hat.tolist()
```

### <a name="create-configuration-file"></a>Creación de un archivo de configuración

Cree un archivo de configuración de implementación y especifique el número de CPU y gigabytes de RAM necesario para el contenedor de ACI. Aunque depende de su modelo, el valor predeterminado de 1 núcleo y 1 gigabyte de RAM suele ser suficiente para muchos modelos. Si más adelante cree que necesita más, tendrá que volver a crear la imagen y volver a implementar el servicio.


```python
from azureml.core.webservice import AciWebservice

aciconfig = AciWebservice.deploy_configuration(cpu_cores=1, 
                                               memory_gb=1, 
                                               tags={"data": "MNIST",  "method" : "sklearn"}, 
                                               description='Predict MNIST with sklearn')
```

### <a name="deploy-in-aci"></a>Implementación en ACI
Tiempo estimado para completar el tutorial: **entre 2 y 5 minutos**

Configuración de la imagen e implementación. El código siguiente realiza estos pasos:

1. Crea el objeto de entorno que contiene las dependencias que necesita el modelo mediante el entorno (`tutorial-env`) guardado durante el entrenamiento.
1. Crea una configuración de inferencia necesaria para implementar el modelo como un servicio web mediante:
   * El archivo de puntuación (`score.py`).
   * objeto de entorno creado en el paso anterior
1. Implementa el modelo en el contenedor de ACI.
1. Obtenga el punto de conexión HTTP del servicio web.


```python
%%time
from azureml.core.webservice import Webservice
from azureml.core.model import InferenceConfig
from azureml.core.environment import Environment
from azureml.core import Workspace
from azureml.core.model import Model

ws = Workspace.from_config()
model = Model(ws, 'sklearn_mnist')


myenv = Environment.get(workspace=ws, name="tutorial-env", version="1")
inference_config = InferenceConfig(entry_script="score.py", environment=myenv)

service = Model.deploy(workspace=ws, 
                       name='sklearn-mnist-svc3', 
                       models=[model], 
                       inference_config=inference_config, 
                       deployment_config=aciconfig)

service.wait_for_deployment(show_output=True)
```

Obtenga punto de conexión HTTP del servicio web de puntuación, que acepta llamadas de cliente REST. Este punto de conexión se puede compartir con todo aquel que desee probar el servicio web, o se puede integrar en una aplicación.


```python
print(service.scoring_uri)
```

## <a name="test-the-model"></a>Prueba del modelo


### <a name="download-test-data"></a>Descarga de los datos de prueba
Descargue los datos de prueba en el directorio **./data/** .


```python
import os
from azureml.core import Dataset
from azureml.opendatasets import MNIST

data_folder = os.path.join(os.getcwd(), 'data')
os.makedirs(data_folder, exist_ok=True)

mnist_file_dataset = MNIST.get_file_dataset()
mnist_file_dataset.download(data_folder, overwrite=True)
```

### <a name="load-test-data"></a>Carga de datos de prueba

Cargue los datos de prueba del directorio **./data/** creado durante el tutorial de aprendizaje.


```python
from utils import load_data
import os
import glob

data_folder = os.path.join(os.getcwd(), 'data')
# note we also shrink the intensity values (X) from 0-255 to 0-1. This helps the neural network converge faster
X_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-images-idx3-ubyte.gz"), recursive=True)[0], False) / 255.0
y_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-labels-idx1-ubyte.gz"), recursive=True)[0], True).reshape(-1)
```

### <a name="predict-test-data"></a>Predicción de los datos de prueba

Inserte el conjunto de datos de prueba en el modelo para obtener predicciones.


El código siguiente realiza estos pasos:
1. Envía los datos como una matriz JSON al servicio web hospedado en ACI. 

1. Usa la API `run` del SDK para invocar el servicio. También se pueden realizar llamadas sin procesar con cualquier herramienta HTTP como curl.


```python
import json
test = json.dumps({"data": X_test.tolist()})
test = bytes(test, encoding='utf8')
y_hat = service.run(input_data=test)
```

###  <a name="examine-the-confusion-matrix"></a>Examen de la matriz de confusión

Genere una matriz de confusión para cuántas muestras del conjunto de prueba se han clasificado correctamente. Observe el valor clasificado incorrectamente de las predicciones incorrectas.


```python
from sklearn.metrics import confusion_matrix

conf_mx = confusion_matrix(y_test, y_hat)
print(conf_mx)
print('Overall accuracy:', np.average(y_hat == y_test))
```

La salida muestra la matriz de confusión:

```output
[[ 960    0    1    2    1    5    6    3    1    1]
 [   0 1112    3    1    0    1    5    1   12    0]
 [   9    8  920   20   10    4   10   11   37    3]
 [   4    0   17  921    2   21    4   12   20    9]
 [   1    2    5    3  915    0   10    2    6   38]
 [  10    2    0   41   10  770   17    7   28    7]
 [   9    3    7    2    6   20  907    1    3    0]
 [   2    7   22    5    8    1    1  950    5   27]
 [  10   15    5   21   15   27    7   11  851   12]
 [   7    8    2   13   32   13    0   24   12  898]]
Overall accuracy: 0.9204
```

Use `matplotlib` para mostrar la matriz de confusión como un gráfico. En este gráfico, el eje X representa los valores reales y el eje Y representa los valores de predicción. El color de cada cuadrícula representa la tasa de errores. Cuanto más claro sea el color, mayor es la tasa de errores. Por ejemplo, muchos 5 se han clasificado incorrectamente como 3. Por lo tanto, verá una cuadrícula brillante en (5,3).

```python
# normalize the diagonal cells so that they don't overpower the rest of the cells when visualized
row_sums = conf_mx.sum(axis=1, keepdims=True)
norm_conf_mx = conf_mx / row_sums
np.fill_diagonal(norm_conf_mx, 0)

fig = plt.figure(figsize=(8, 5))
ax = fig.add_subplot(111)
cax = ax.matshow(norm_conf_mx, cmap=plt.cm.bone)
ticks = np.arange(0, 10, 1)
ax.set_xticks(ticks)
ax.set_yticks(ticks)
ax.set_xticklabels(ticks)
ax.set_yticklabels(ticks)
fig.colorbar(cax)
plt.ylabel('true labels', fontsize=14)
plt.xlabel('predicted values', fontsize=14)
plt.savefig('conf.png')
plt.show()
```

![Gráfico en el que se muestra la matriz de confusión](./media/tutorial-deploy-models-with-aml/confusion.png)


## <a name="show-predictions"></a>Presentación de las predicciones

Pruebe el modelo implementado con una muestra aleatoria de 30 imágenes de los datos de prueba.  


1. Imprime las predicciones devueltas y las representa junto con las imágenes de entrada. Se usa un color de fuente rojo e imagen inversa (blanco sobre negro) para resaltar los ejemplos clasificados incorrectamente. 

 Como la precisión del modelo es alta, es posible que deba ejecutar el siguiente código varias veces para que pueda ver un ejemplo de clasificación incorrecta.



```python
import json

# find 30 random samples from test set
n = 30
sample_indices = np.random.permutation(X_test.shape[0])[0:n]

test_samples = json.dumps({"data": X_test[sample_indices].tolist()})
test_samples = bytes(test_samples, encoding='utf8')

# predict using the deployed model
result = service.run(input_data=test_samples)

# compare actual value vs. the predicted values:
i = 0
plt.figure(figsize = (20, 1))

for s in sample_indices:
    plt.subplot(1, n, i + 1)
    plt.axhline('')
    plt.axvline('')
    
    # use different color for misclassified sample
    font_color = 'red' if y_test[s] != result[i] else 'black'
    clr_map = plt.cm.gray if y_test[s] != result[i] else plt.cm.Greys
    
    plt.text(x=10, y=-10, s=result[i], fontsize=18, color=font_color)
    plt.imshow(X_test[s].reshape(28, 28), cmap=clr_map)
    
    i = i + 1
plt.show()
```

También puede enviar la solicitud HTTP sin procesar para probar el servicio web.


```python
import requests

# send a random row from the test set to score
random_index = np.random.randint(0, len(X_test)-1)
input_data = "{\"data\": [" + str(list(X_test[random_index])) + "]}"

headers = {'Content-Type': 'application/json'}

# for AKS deployment you'd need to the service key in the header as well
# api_key = service.get_key()
# headers = {'Content-Type':'application/json',  'Authorization':('Bearer '+ api_key)} 

resp = requests.post(service.scoring_uri, input_data, headers=headers)

print("POST to url", service.scoring_uri)
#print("input data:", input_data)
print("label:", y_test[random_index])
print("prediction:", resp.text)
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para conservar el grupo de recursos y el área de trabajo para otros tutoriales y para su exploración, puede eliminar solo la implementación de Container Instances con esta llamada API:

```python
service.delete()
```

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]


## <a name="next-steps"></a>Pasos siguientes

+ Más información sobre todas las [opciones de implementación de Azure Machine Learning](how-to-deploy-and-where.md).
+ Obtenga información sobre cómo [crear clientes para el servicio web](how-to-consume-web-service.md).
+  [Realización de predicciones sobre grandes cantidades de datos](how-to-use-parallel-run-step.md) asincrónicamente.
+ Supervise los modelos de Azure Machine Learning con [Application Insights](how-to-enable-app-insights.md).
+ Consulte el tutorial sobre [selección automática del algoritmo](tutorial-auto-train-models.md). 
