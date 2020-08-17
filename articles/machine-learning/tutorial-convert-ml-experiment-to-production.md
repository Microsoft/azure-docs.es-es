---
title: Conversión del código de cuaderno en scripts de Python
titleSuffix: Azure Machine Learning
description: Convierta los cuadernos experimentales de aprendizaje automático en código listo para producción mediante la plantilla de código MLOpsPython. Después, puede probar, implementar y automatizar ese código.
author: bjcmit
ms.author: brysmith
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.date: 04/30/2020
ms.custom: devx-track-python
ms.openlocfilehash: 3a6ce5860704e6fd16b79fc253650dd45ec743e7
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852623"
---
# <a name="tutorial-convert-ml-experiments-to-production-python-code"></a>Tutorial: Conversión de experimentos de ML en código de producción de Python

En este tutorial, aprenderá a convertir cuadernos de Juptyer en scripts de Python para facilitar las pruebas y la automatización mediante la plantilla de código MLOpsPython y Azure Machine Learning. Normalmente, este proceso se usa para tomar código de experimentación o entrenamiento de un cuaderno de Juptyer y convertirlo en scripts de Python. Después, esos scripts se pueden usar para realizar pruebas y automatización de CI/CD en el entorno de producción. 

Un proyecto de aprendizaje automático requiere experimentación al probar hipótesis con herramientas ágiles, como Jupyter Notebook, con conjuntos de datos auténticos. Una vez que el modelo está listo para producción, el código del modelo debe colocarse en un repositorio de código de producción. En algunos casos, el código del modelo se debe convertir en scripts de Python que se colocarán en el repositorio de código de producción. En este tutorial se describe un enfoque recomendado sobre cómo exportar código de experimentación a scripts de Python.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Limpiar código no esencial
> * Refactorizar código de Jupyter Notebook en funciones
> * Crear scripts de Python para tareas relacionadas
> * Crear pruebas unitarias

## <a name="prerequisites"></a>Prerrequisitos

- Genere la [plantilla MLOpsPython](https://github.com/microsoft/MLOpsPython/generate) y use los cuadernos `experimentation/Diabetes Ridge Regression Training.ipynb` y `experimentation/Diabetes Ridge Regression Scoring.ipynb`. Estos cuadernos se utilizan como ejemplo de conversión de experimentación en producción. Estos cuadernos se pueden encontrar en [https://github.com/microsoft/MLOpsPython/tree/master/experimentation](https://github.com/microsoft/MLOpsPython/tree/master/experimentation).
- Instalar `nbconvert`. Siga únicamente las instrucciones de instalación de la sección de __instalación de nbconvert__ en la página de [instalación](https://nbconvert.readthedocs.io/en/latest/install.html).

## <a name="remove-all-nonessential-code"></a>Eliminación de todo el código no esencial

Parte del código escrito durante la experimentación solo tiene fines de exploración. Por lo tanto, el primer paso para convertir código experimental en código de producción es quitar este código no esencial. La eliminación del código no esencial también hará que el código sea más fácil de mantener. En esta sección, quitará el código del cuaderno `experimentation/Diabetes Ridge Regression Training.ipynb`. Las instrucciones que imprimen la forma de `X` y `y` y la celda que llama a `features.describe` son solo para la exploración de datos y se pueden quitar. Después de quitar el código no esencial, `experimentation/Diabetes Ridge Regression Training.ipynb` debe tener un aspecto similar al código siguiente sin marcado:

```python
from sklearn.datasets import load_diabetes
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split
import joblib
import pandas as pd

sample_data = load_diabetes()

df = pd.DataFrame(
    data=sample_data.data,
    columns=sample_data.feature_names)
df['Y'] = sample_data.target

X = df.drop('Y', axis=1).values
y = df['Y'].values

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=0)
data = {"train": {"X": X_train, "y": y_train},
        "test": {"X": X_test, "y": y_test}}

args = {
    "alpha": 0.5
}

reg_model = Ridge(**args)
reg.fit(data["train"]["X"], data["train"]["y"])

preds = reg_model.predict(data["test"]["X"])
mse = mean_squared_error(preds, y_test)
metrics = {"mse": mse}
print(metrics)

model_name = "sklearn_regression_model.pkl"
joblib.dump(value=reg, filename=model_name)
```

## <a name="refactor-code-into-functions"></a>Refactorización del código en funciones

En segundo lugar, el código de Jupyter debe refactorizarse en funciones. La refactorización del código en funciones facilita la realización de pruebas unitarias y hace que el código sea más sencillo de mantener. En esta sección, refactorizará lo siguiente:

- El cuaderno Diabetes Ridge Regression Training (`experimentation/Diabetes Ridge Regression Training.ipynb`)
- El cuaderno Diabetes Ridge Regression Scoring (`experimentation/Diabetes Ridge Regression Scoring.ipynb`)

### <a name="refactor-diabetes-ridge-regression-training-notebook-into-functions"></a>Refactorización del cuaderno Diabetes Ridge Regression Training en funciones

En `experimentation/Diabetes Ridge Regression Training.ipynb`, complete los pasos siguientes:

1. Cree una función llamada `split_data` para dividir la trama de datos en datos de prueba y de entrenamiento. La función debe tomar la trama de datos `df` como parámetro y devolver un diccionario que contenga las claves `train` y `test`.

    Mueva el código del encabezado *Split Data into Training and Validation Sets* a la función `split_data` y modifíquelo para que devuelva el objeto `data`.

1. Cree una función llamada `train_model`, que toma los parámetros `data` y `args`, y devuelve un modelo entrenado.

    Mueva el código del encabezado *Training Model on Training Set* a la función `train_model` y modifíquelo para que devuelva el objeto `reg_model`. Quite el diccionario `args`, los valores procederán del parámetro `args`.

1. Cree una función llamada `get_model_metrics`, que toma los parámetros `reg_model` y `data`, y evalúa el modelo y, a continuación, devuelve un diccionario de métricas para el modelo entrenado.

    Mueva el código del encabezado *Validate Model on Validation Set* a la función `get_model_metrics` y modifíquelo para que devuelva el objeto `metrics`.

Las tres funciones deben quedar como se observa a continuación:

```python
# Split the dataframe into test and train data
def split_data(df):
    X = df.drop('Y', axis=1).values
    y = df['Y'].values

    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=0)
    data = {"train": {"X": X_train, "y": y_train},
            "test": {"X": X_test, "y": y_test}}
    return data


# Train the model, return the model
def train_model(data, args):
    reg_model = Ridge(**args)
    reg_model.fit(data["train"]["X"], data["train"]["y"])
    return reg_model


# Evaluate the metrics for the model
def get_model_metrics(reg_model, data):
    preds = reg_model.predict(data["test"]["X"])
    mse = mean_squared_error(preds, data["test"]["y"])
    metrics = {"mse": mse}
    return metrics
```

Continuando en el archivo `experimentation/Diabetes Ridge Regression Training.ipynb`, complete los pasos siguientes:

1. Cree una nueva función llamada `main`, que no toma parámetros y no devuelve nada.
1. Copie el código que se encuentra en el encabezado "Load Data" a la función `main`.
1. Agregue invocaciones para las funciones recién escritas en la función `main`:
    ```python
    # Split Data into Training and Validation Sets
    data = split_data(df)
    ```

    ```python
    # Train Model on Training Set
    args = {
        "alpha": 0.5
    }
    reg = train_model(data, args)
    ```

    ```python
    # Validate Model on Validation Set
    metrics = get_model_metrics(reg, data)
    ```
1. Mueva el código que se encuentra en el encabezado "Save Model" en la función `main`.

La función `main` debe ser similar al código siguiente:

```python
def main():
    # Load Data
    sample_data = load_diabetes()

    df = pd.DataFrame(
        data=sample_data.data,
        columns=sample_data.feature_names)
    df['Y'] = sample_data.target

    # Split Data into Training and Validation Sets
    data = split_data(df)

    # Train Model on Training Set
    args = {
        "alpha": 0.5
    }
    reg = train_model(data, args)

    # Validate Model on Validation Set
    metrics = get_model_metrics(reg, data)

    # Save Model
    model_name = "sklearn_regression_model.pkl"

    joblib.dump(value=reg, filename=model_name)
```

En esta fase, no debería haber quedado ningún código en el cuaderno que no esté en una función, excepto las instrucciones Import de la primera celda.

Agregue una instrucción que llame a la función `main`.

```python
main()
```

Después de la refactorización, `experimentation/Diabetes Ridge Regression Training.ipynb` debe ser similar al código siguiente sin el marcado:

```python
from sklearn.datasets import load_diabetes
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split
import pandas as pd
import joblib


# Split the dataframe into test and train data
def split_data(df):
    X = df.drop('Y', axis=1).values
    y = df['Y'].values

    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=0)
    data = {"train": {"X": X_train, "y": y_train},
            "test": {"X": X_test, "y": y_test}}
    return data


# Train the model, return the model
def train_model(data, args):
    reg_model = Ridge(**args)
    reg_model.fit(data["train"]["X"], data["train"]["y"])
    return reg_model


# Evaluate the metrics for the model
def get_model_metrics(reg_model, data):
    preds = reg_model.predict(data["test"]["X"])
    mse = mean_squared_error(preds, data["test"]["y"])
    metrics = {"mse": mse}
    return metrics


def main():
    # Load Data
    sample_data = load_diabetes()

    df = pd.DataFrame(
        data=sample_data.data,
        columns=sample_data.feature_names)
    df['Y'] = sample_data.target

    # Split Data into Training and Validation Sets
    data = split_data(df)

    # Train Model on Training Set
    args = {
        "alpha": 0.5
    }
    reg = train_model(data, args)

    # Validate Model on Validation Set
    metrics = get_model_metrics(reg, data)

    # Save Model
    model_name = "sklearn_regression_model.pkl"

    joblib.dump(value=reg, filename=model_name)

main()
```

### <a name="refactor-diabetes-ridge-regression-scoring-notebook-into-functions"></a>Refactorización del cuaderno Diabetes Ridge Regression Scoring en funciones

En `experimentation/Diabetes Ridge Regression Scoring.ipynb`, complete los pasos siguientes:

1. Cree una nueva función llamada `init`, que no toma parámetros y no devuelve nada.
1. Copie el código que se encuentra bajo del encabezado "Load Model" en la función `init`.

La función `init` debe ser similar al código siguiente:

```python
def init():
    model_path = Model.get_model_path(
        model_name="sklearn_regression_model.pkl")
    model = joblib.load(model_path)
```

Una vez creada la función `init`, reemplace todo el código que se encuentra bajo del encabezado "Load Model" por una única llamada a `init`, como se indica a continuación:

```python
init()
```

En `experimentation/Diabetes Ridge Regression Scoring.ipynb`, complete los pasos siguientes:

1. Cree una nueva función llamada `run`, que toma `raw_data` y `request_headers` como parámetros y devuelve un diccionario de resultados como se indica a continuación:

    ```python
    {"result": result.tolist()}
    ```

1. Copie el código de los encabezados "Prepare Data" y "Score Data" en la función `run`.

    La función `run` debe ser similar al código siguiente (recuerde quitar las instrucciones que establecen las variables `raw_data` y `request_headers`, que se usarán más adelante cuando se llame a la función `run`):

    ```python
    def run(raw_data, request_headers):
        data = json.loads(raw_data)["data"]
        data = numpy.array(data)
        result = model.predict(data)

        return {"result": result.tolist()}
    ```

Una vez creada la función `run`, reemplace todo el código de los encabezados "Prepare Data" y "Score Data" por el siguiente:

```python
raw_data = '{"data":[[1,2,3,4,5,6,7,8,9,10],[10,9,8,7,6,5,4,3,2,1]]}'
request_header = {}
prediction = run(raw_data, request_header)
print("Test result: ", prediction)
```

El código anterior establece las variables `raw_data` y `request_header`, llama a la función `run` con `raw_data` y `request_header` e imprime las predicciones.

Después de la refactorización, `experimentation/Diabetes Ridge Regression Scoring.ipynb` debe ser similar al código siguiente sin el marcado:

```python
import json
import numpy
from azureml.core.model import Model
import joblib

def init():
    model_path = Model.get_model_path(
        model_name="sklearn_regression_model.pkl")
    model = joblib.load(model_path)

def run(raw_data, request_headers):
    data = json.loads(raw_data)["data"]
    data = numpy.array(data)
    result = model.predict(data)

    return {"result": result.tolist()}

init()
test_row = '{"data":[[1,2,3,4,5,6,7,8,9,10],[10,9,8,7,6,5,4,3,2,1]]}'
request_header = {}
prediction = run(test_row, {})
print("Test result: ", prediction)
```

## <a name="combine-related-functions-in-python-files"></a>Combinación de funciones relacionadas en archivos de Python

En tercer lugar, las funciones relacionadas deben combinarse en archivos de Python para mejorar la reutilización del código. En esta sección, creará archivos de Python para los siguientes cuadernos:

- El cuaderno Diabetes Ridge Regression Training (`experimentation/Diabetes Ridge Regression Training.ipynb`)
- El cuaderno Diabetes Ridge Regression Scoring (`experimentation/Diabetes Ridge Regression Scoring.ipynb`)

### <a name="create-python-file-for-the-diabetes-ridge-regression-training-notebook"></a>Creación de archivo de Python para el cuaderno Diabetes Ridge Regression Training

Convierta el cuaderno en un script ejecutable mediante la ejecución de la siguiente instrucción en un símbolo del sistema, que usa el paquete `nbconvert` y la ruta de acceso de `experimentation/Diabetes Ridge Regression Training.ipynb`:

```
jupyter nbconvert -- to script "Diabetes Ridge Regression Training.ipynb" –output train
```

Una vez convertido el cuaderno en `train.py`, quite todos los comentarios innecesarios. Reemplace la llamada a `main()` al final del archivo por una invocación condicional como el código siguiente:

```python
if __name__ == '__main__':
    main()
```

El archivo `train.py` debe ser similar al siguiente código:

```python
from sklearn.datasets import load_diabetes
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split
import pandas as pd
import joblib


# Split the dataframe into test and train data
def split_data(df):
    X = df.drop('Y', axis=1).values
    y = df['Y'].values

    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=0)
    data = {"train": {"X": X_train, "y": y_train},
            "test": {"X": X_test, "y": y_test}}
    return data


# Train the model, return the model
def train_model(data, args):
    reg_model = Ridge(**args)
    reg_model.fit(data["train"]["X"], data["train"]["y"])
    return reg_model


# Evaluate the metrics for the model
def get_model_metrics(reg_model, data):
    preds = reg_model.predict(data["test"]["X"])
    mse = mean_squared_error(preds, data["test"]["y"])
    metrics = {"mse": mse}
    return metrics


def main():
    # Load Data
    sample_data = load_diabetes()

    df = pd.DataFrame(
        data=sample_data.data,
        columns=sample_data.feature_names)
    df['Y'] = sample_data.target

    # Split Data into Training and Validation Sets
    data = split_data(df)

    # Train Model on Training Set
    args = {
        "alpha": 0.5
    }
    reg = train_model(data, args)

    # Validate Model on Validation Set
    metrics = get_model_metrics(reg, data)

    # Save Model
    model_name = "sklearn_regression_model.pkl"

    joblib.dump(value=reg, filename=model_name)

if __name__ == '__main__':
    main()
```

`train.py` se puede invocar ahora desde un terminal mediante la ejecución de `python train.py`.
También se puede llamar a las funciones de `train.py` desde otros archivos.

El archivo `train_aml.py` que se encuentra en el directorio `diabetes_regression/training` en el repositorio de MLOpsPython llama a las funciones definidas en `train.py` en el contexto de una ejecución del experimento de Azure Machine Learning. También se puede llamar a las funciones en pruebas unitarias, que se describen más adelante en esta guía.

### <a name="create-python-file-for-the-diabetes-ridge-regression-scoring-notebook"></a>Creación de archivo de Python para el cuaderno Diabetes Ridge Regression Scoring

Convierta el cuaderno en un script ejecutable mediante la ejecución de la siguiente instrucción en un símbolo del sistema, que usa el paquete `nbconvert` y la ruta de acceso de `experimentation/Diabetes Ridge Regression Scoring.ipynb`:

```
jupyter nbconvert -- to script "Diabetes Ridge Regression Scoring.ipynb" –output score
```

Una vez convertido el cuaderno en `score.py`, quite todos los comentarios innecesarios. El archivo `score.py` debe ser similar al siguiente código:

```python
import json
import numpy
from azureml.core.model import Model
import joblib

def init():
    model_path = Model.get_model_path(
        model_name="sklearn_regression_model.pkl")
    model = joblib.load(model_path)

def run(raw_data, request_headers):
    data = json.loads(raw_data)["data"]
    data = numpy.array(data)
    result = model.predict(data)

    return {"result": result.tolist()}

init()
test_row = '{"data":[[1,2,3,4,5,6,7,8,9,10],[10,9,8,7,6,5,4,3,2,1]]}'
request_header = {}
prediction = run(test_row, request_header)
print("Test result: ", prediction)
```

La variable `model` tiene que ser global para que sea visible en todo el script. Agregue la siguiente instrucción al principio de la función `init`:

```python
global model
```

Después de agregar la instrucción anterior, la función `init` debe ser similar al código siguiente:

```python
def init():
    global model

    # load the model from file into a global object
    model_path = Model.get_model_path(
        model_name="sklearn_regression_model.pkl")
    model = joblib.load(model_path)
```

## <a name="create-unit-tests-for-each-python-file"></a>Creación de pruebas unitarias para cada archivo de Python

En cuarto lugar, cree pruebas unitarias para las funciones de Python. Las pruebas unitarias protegen el código de las regresiones funcionales y facilitan el mantenimiento. En esta sección, creará pruebas unitarias para las funciones de `train.py`.

`train.py` contiene varias funciones, pero solo se creará una prueba unitaria para la función `train_model` mediante el marco Pytest en este tutorial. Pytest no es el único marco de pruebas unitarias de Python, pero es uno de los más usados. Para más información, visite [Pytest](https://pytest.org).

Una prueba unitaria normalmente contiene tres acciones principales:

- Organizar el objeto: crear y configurar los objetos necesarios
- Actuar sobre un objeto
- Declarar lo que se espera

La prueba unitaria llamará a `train_model` con algunos datos y argumentos codificados de forma rígida, y validará que `train_model` ha actuado según lo previsto mediante el modelo entrenado resultante para crear una predicción y comparar esa predicción con un valor esperado.

```python
import numpy as np
from code.training.train import train_model


def test_train_model():
    # Arrange
    X_train = np.array([1, 2, 3, 4, 5, 6]).reshape(-1, 1)
    y_train = np.array([10, 9, 8, 8, 6, 5])
    data = {"train": {"X": X_train, "y": y_train}}

    # Act
    reg_model = train_model(data, {"alpha": 1.2})

    # Assert
    preds = reg_model.predict([[1], [2]])
    np.testing.assert_almost_equal(preds, [9.93939393939394, 9.03030303030303])
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo convertir de un código de experimento a uno de producción, consulte los siguientes vínculos para más información y para los pasos siguientes:

+ [MLOpsPython](https://github.com/microsoft/MLOpsPython/blob/master/docs/custom_model.md): Cree una canalización de CI/CD para entrenar, evaluar e implementar su propio modelo mediante Azure Pipelines y Azure Machine Learning
+ [Supervisión de métricas y ejecuciones de experimentos de Azure ML](https://docs.microsoft.com/azure/machine-learning/how-to-track-experiments)
+ [Supervisión y recopilación de datos de los puntos de conexión del servicio web ML](https://docs.microsoft.com/azure/machine-learning/how-to-enable-app-insights)
