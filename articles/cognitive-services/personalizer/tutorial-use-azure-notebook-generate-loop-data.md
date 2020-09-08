---
title: 'Tutorial: Personalizer en Azure Notebook'
titleSuffix: Azure Cognitive Services
description: En este tutorial se simula un bucle de Personalizer en un proyecto de Azure Notebook que sugiere qué tipo de café debe solicitar un cliente. Los usuarios y sus preferencias están almacenados en un conjunto de datos de usuario. La información sobre el café también está disponible y almacenada en un conjunto de datos de café.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: tutorial
ms.date: 04/27/2020
ms.custom: devx-track-python
ms.openlocfilehash: 0cf2aa504fd7a36a55740b3a59eeb3759e67f469
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88509916"
---
# <a name="tutorial-use-personalizer-in-azure-notebook"></a>Tutorial: Uso de Personalizer en Azure Notebook

En este tutorial se ejecuta un bucle de Personalizer en una instancia de Azure Notebook y se muestra el ciclo de vida completo de uno de estos bucles.

El bucle sugiere qué tipo de café debe solicitar un cliente. Los usuarios y sus preferencias están almacenados en un conjunto de datos de usuario. La información sobre el café está almacenada en un conjunto de datos de café.

## <a name="users-and-coffee"></a>Usuarios y café

El cuaderno, que simula una interacción del usuario con un sitio web, selecciona una combinación aleatoria de usuario, hora del día e información meteorológica del conjunto de datos. A continuación se muestra un resumen de la información de usuario:

|Clientes: características de contexto|Horas del día|Tipos de condición meteorológica|
|--|--|--|
|Alice<br>Bob<br>Cathy<br>Dave|Mañana<br>Tarde<br>Noche|Soleado<br>Lluvioso<br>Nevado|

Para ayudar a que Personalizer aprenda, con el tiempo, el _sistema_ también conoce detalles sobre la selección de café de cada persona.

|Café: características de acción|Tipos de temperatura|Lugares de origen|Tipos de tueste|Orgánico|
|--|--|--|--|--|
|Cappucino|Acceso frecuente|Kenia|Oscuro|Orgánico|
|Infusión fría|Frío|Brasil|Ligero|Orgánico|
|Moca con hielo|Frío|Etiopía|Ligero|No orgánico|
|Latte|Acceso frecuente|Brasil|Oscuro|No orgánico|

La **finalidad** del bucle de Personalizer es encontrar la mejor correspondencia entre los usuarios y el café tantas veces como sea posible.

El código de este tutorial está disponible en el [repositorio de ejemplos de GitHub para Personalizer](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/tree/master/samples/azurenotebook).

## <a name="how-the-simulation-works"></a>Funcionamiento de la simulación

Cuando el sistema comienza a funcionar, las sugerencias de Personalizer solo aciertan entre el 20 y el 30 % de las veces. Este éxito se indica mediante la recompensa que se envía de vuelta a la API de recompensas de Personalizer, con una puntuación de 1. Después de algunas llamadas a Rank and Reward, el sistema mejora.

Después de las solicitudes iniciales, ejecute una evaluación sin conexión. Esto permite que Personalizer revise los datos y sugiera una directiva de aprendizaje mejor. Aplique la nueva directiva de aprendizaje y vuelva a ejecutar el cuaderno con un 20 % del total de solicitudes anteriores. El bucle funcionará mejor con la nueva directiva de aprendizaje.

## <a name="rank-and-reward-calls"></a>Llamadas a Rank y Reward

Para cada una de las pocas miles de llamadas al servicio Personalizer, Azure Notebook envía la solicitud **Rank** a la API REST:

* Un identificador único para el evento Rank/Reward
* Características de contexto: una combinación aleatoria de usuario, condición meteorológica y hora del día que simula a un usuario en un sitio web o un dispositivo móvil
* Acciones con características: _todos_ los datos de café a partir de los cuales Personalizer realiza una sugerencia

El sistema recibe la solicitud y compara esa predicción con la opción conocida del usuario para la misma hora del día y condición meteorológica. Si la opción conocida es la misma que la opción de la predicción, se devuelve a Personalizer un valor de **Reward** de 1. En caso contrario, la recompensa que se envía de vuelta es 0.

> [!Note]
> Se trata de una simulación, por lo que el algoritmo de la recompensa es sencillo. En un escenario real, para determinar la puntuación de la recompensa el algoritmo deberá utilizar lógica de negocios, posiblemente con pesos para los diversos aspectos de la experiencia del cliente.


## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta de [Azure Notebook](https://notebooks.azure.com/).
* Un [recurso Personalizer de Azure](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesPersonalizer).
    * Si ya ha usado el recurso Personalizer, asegúrese de que [borra los datos](how-to-settings.md#clear-data-for-your-learning-loop) del recurso en Azure Portal.
* Cargue todos los archivos de [este ejemplo](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/tree/master/samples/azurenotebook) en un proyecto de Azure Notebook.

Descripciones de archivo:

* [Personalizator.ipynb](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/blob/master/samples/azurenotebook/Personalizer.ipynb) es el cuaderno de Jupyter Notebook para este tutorial.
* El [conjunto de datos de usuario](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/blob/master/samples/azurenotebook/users.json) está almacenado en un objeto JSON.
* El [conjunto de datos de café](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/blob/master/samples/azurenotebook/coffee.json) está almacenado en un objeto JSON.
* El [JSON de la solicitud de ejemplo](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/blob/master/samples/azurenotebook/example-rankrequest.json) tiene el formato esperado para una solicitud POST a la API Rank.

## <a name="configure-personalizer-resource"></a>Configuración del recurso Personalizer

En Azure Portal, configure el [recurso Personalizer](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesPersonalizer) con el valor de **Frecuencia de actualización del modelo** establecido en 15 segundos y el valor de **Tiempo de espera de recompensa** establecido también en 15 segundos. Estos valores se encuentran en la página **[Configuración](how-to-settings.md#configure-service-settings-in-the-azure-portal)** .

|Configuración|Value|
|--|--|
|Frecuencia de actualización del modelo|15 segundos|
|Tiempo de espera de recompensa|15 segundos|

Estos valores tienen una duración muy corta para mostrar los cambios en este tutorial. No se deben usar en un escenario de producción sin comprobar que logran el objetivo buscado con el bucle de Personalizer.

## <a name="set-up-the-azure-notebook"></a>Configuración de Azure Notebook

1. Cambie el kernel a `Python 3.6`.
1. Abra el archivo `Personalizer.ipynb` .

## <a name="run-notebook-cells"></a>Ejecución de celdas del cuaderno

Procese cada celda ejecutable y espere a que finalice. Sabrá que ha terminado cuando los corchetes que aparecen junto a la celda muestren un número en lugar de `*`. En las secciones siguientes se explica qué hace cada celda de programación y qué esperar como salida.

### <a name="include-the-python-modules"></a>Inclusión de los módulos de Python

Incluya los módulos de Python necesarios. La celda no tiene ninguna salida.

```python
import json
import matplotlib.pyplot as plt
import random
import requests
import time
import uuid
```

### <a name="set-personalizer-resource-key-and-name"></a>Establecimiento de la clave y el nombre del recurso Personalizer

En Azure Portal, busque la clave y el punto de conexión en la página **Inicio rápido** del recurso Personalizer. Cambie el valor de `<your-resource-name>` al nombre del recurso Personalizer. Cambie el valor de `<your-resource-key>` a la clave de Personalizer.

```python
# Replace 'personalization_base_url' and 'resource_key' with your valid endpoint values.
personalization_base_url = "https://<your-resource-name>.cognitiveservices.azure.com/"
resource_key = "<your-resource-key>"
```

### <a name="print-current-date-and-time"></a>Impresión de la fecha y la hora actuales
Use esta función para anotar las horas de inicio y finalización de la función iterativa, iterations.

Estas celdas no tienen ninguna salida. Cuando se llama a la función, la salida indica la fecha y hora actuales.

```python
# Print out current datetime
def currentDateTime():
    currentDT = datetime.datetime.now()
    print (str(currentDT))
```

### <a name="get-the-last-model-update-time"></a>Obtención de la última hora de actualización del modelo

Cuando se llama a la función, `get_last_updated`, esta imprime la fecha y la hora de la última actualización del modelo.

Estas celdas no tienen ninguna salida. Cuando se llama a la función, la salida indica la última fecha de entrenamiento del modelo.

La función usa una API REST GET para [obtener las propiedades del modelo](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/GetModelProperties).

```python
# ititialize variable for model's last modified date
modelLastModified = ""
```

```python
def get_last_updated(currentModifiedDate):

    print('-----checking model')

    # get model properties
    response = requests.get(personalization_model_properties_url, headers = headers, params = None)

    print(response)
    print(response.json())

    # get lastModifiedTime
    lastModifiedTime = json.dumps(response.json()["lastModifiedTime"])

    if (currentModifiedDate != lastModifiedTime):
        currentModifiedDate = lastModifiedTime
        print(f'-----model updated: {lastModifiedTime}')
```

### <a name="get-policy-and-service-configuration"></a>Obtención de la configuración de la directiva y el servicio

Compruebe el estado del servicio con estas dos llamadas a REST.

Estas celdas no tienen ninguna salida. Cuando se llama a la función, la salida indica los valores del servicio.

```python
def get_service_settings():

    print('-----checking service settings')

    # get learning policy
    response = requests.get(personalization_model_policy_url, headers = headers, params = None)

    print(response)
    print(response.json())

    # get service settings
    response = requests.get(personalization_service_configuration_url, headers = headers, params = None)

    print(response)
    print(response.json())
```

### <a name="construct-urls-and-read-json-data-files"></a>Construcción de las direcciones URL y lectura de los archivos de datos JSON

Esta celda realiza lo siguiente:

* compila las direcciones URL usadas en las llamadas a REST;
* establece el encabezado de seguridad mediante la clave del recurso Personalizer;
* establece la inicialización aleatoria para el identificador del evento Rank;
* lee los archivos de datos JSON;
* llama al método `get_last_updated` (se ha quitado la directiva de aprendizaje de la salida del ejemplo);
* llama al método `get_service_settings`.

La celda tiene la salida de la llamada a las funciones `get_last_updated` y `get_service_settings`.

```python
# build URLs
personalization_rank_url = personalization_base_url + "personalizer/v1.0/rank"
personalization_reward_url = personalization_base_url + "personalizer/v1.0/events/" #add "{eventId}/reward"
personalization_model_properties_url = personalization_base_url + "personalizer/v1.0/model/properties"
personalization_model_policy_url = personalization_base_url + "personalizer/v1.0/configurations/policy"
personalization_service_configuration_url = personalization_base_url + "personalizer/v1.0/configurations/service"

headers = {'Ocp-Apim-Subscription-Key' : resource_key, 'Content-Type': 'application/json'}

# context
users = "users.json"

# action features
coffee = "coffee.json"

# empty JSON for Rank request
requestpath = "example-rankrequest.json"

# initialize random
random.seed(time.time())

userpref = None
rankactionsjsonobj = None
actionfeaturesobj = None

with open(users) as handle:
    userpref = json.loads(handle.read())

with open(coffee) as handle:
    actionfeaturesobj = json.loads(handle.read())

with open(requestpath) as handle:
    rankactionsjsonobj = json.loads(handle.read())

get_last_updated(modelLastModified)
get_service_settings()

print(f'User count {len(userpref)}')
print(f'Coffee count {len(actionfeaturesobj)}')
```

Compruebe que los valores de `rewardWaitTime` y `modelExportFrequency` de la salida están establecidos en 15 segundos.

```console
-----checking model
<Response [200]>
{'creationTime': '0001-01-01T00:00:00+00:00', 'lastModifiedTime': '0001-01-01T00:00:00+00:00'}
-----model updated: "0001-01-01T00:00:00+00:00"
-----checking service settings
<Response [200]>
{...learning policy...}
<Response [200]>
{'rewardWaitTime': '00:00:15', 'defaultReward': 0.0, 'rewardAggregation': 'earliest', 'explorationPercentage': 0.2, 'modelExportFrequency': '00:00:15', 'logRetentionDays': -1}
User count 4
Coffee count 4
```

### <a name="troubleshooting-the-first-rest-call"></a>Solución de problemas de la primera llamada a REST

La celda anterior es la primera celda que llama a Personalizer. Asegúrese de que el código de estado REST de la salida es `<Response [200]>`. Si recibe un error, como 404, pero está seguro de que la clave y el nombre del recurso son correctos, vuelva a cargar el cuaderno.

Asegúrese de que el número de cafés y de usuarios es 4 en ambos casos. Si recibe un error, compruebe que ha cargado los tres archivos JSON.

### <a name="set-up-metric-chart-in-azure-portal"></a>Configuración de un gráfico de métricas en Azure Portal

Más adelante en este tutorial se puede ver el proceso de larga duración de las 10 000 solicitudes desde el explorador, con un cuadro de texto de actualización. Cuando finalice ese proceso, puede que resulte más fácil verlo en un gráfico o como suma total. Para ver esta información, use las métricas proporcionadas con el recurso. Puede crear el gráfico ahora que ha completado una solicitud al servicio y, después, actualizar el gráfico periódicamente mientras continúa el proceso de larga duración.

1. En Azure Portal, seleccione el recurso Personalizer.
1. En la navegación del recurso, seleccione **Métricas** bajo Supervisión.
1. En el gráfico, seleccione **Agregar métrica**.
1. El espacio de nombres del recurso y la métrica ya están establecidos. Solo tiene que seleccionar la métrica de **Llamadas correctas** y la agregación **Suma**.
1. Cambie el filtro de tiempo a las últimas 4 horas.

    ![Configuración del gráfico de métricas en Azure Portal con incorporación de una métrica para las llamadas correctas de las últimas 4 horas](./media/tutorial-azure-notebook/metric-chart-setting.png)

    El gráfico debería mostrar tres llamadas correctas.

### <a name="generate-a-unique-event-id"></a>Generación de un identificador de evento único

Esta función genera un identificador único para cada llamada a Rank. El identificador se usa para identificar la información de la llamada a Rank y Reward. Este valor puede proceder de un proceso de negocio, como un identificador de vista web o un identificador de transacción.

La celda no tiene ninguna salida. Cuando se llama a la función, la salida indica el identificador único.

```python
def add_event_id(rankjsonobj):
    eventid = uuid.uuid4().hex
    rankjsonobj["eventId"] = eventid
    return eventid
```

### <a name="get-random-user-weather-and-time-of-day"></a>Obtención del usuario, la condición meteorológica y la hora del día aleatorios

Esta función selecciona una combinación única de usuario, condición meteorológica y hora del día, y agrega esos elementos al objeto JSON que se va a enviar a la solicitud Rank.

La celda no tiene ninguna salida. Cuando se llama a la función, devuelve los valores aleatorios de nombre del usuario, condición meteorológica y hora del día.

Se muestra a continuación la lista de los cuatro usuarios y sus preferencias; por brevedad, solo se muestran algunas preferencias:

```json
{
  "Alice": {
    "Sunny": {
      "Morning": "Cold brew",
      "Afternoon": "Iced mocha",
      "Evening": "Cold brew"
    }...
  },
  "Bob": {
    "Sunny": {
      "Morning": "Cappucino",
      "Afternoon": "Iced mocha",
      "Evening": "Cold brew"
    }...
  },
  "Cathy": {
    "Sunny": {
      "Morning": "Latte",
      "Afternoon": "Cold brew",
      "Evening": "Cappucino"
    }...
  },
  "Dave": {
    "Sunny": {
      "Morning": "Iced mocha",
      "Afternoon": "Iced mocha",
      "Evening": "Iced mocha"
    }...
  }
}
```

```python
def add_random_user_and_contextfeatures(namesoption, weatheropt, timeofdayopt, rankjsonobj):
    name = namesoption[random.randint(0,3)]
    weather = weatheropt[random.randint(0,2)]
    timeofday = timeofdayopt[random.randint(0,2)]
    rankjsonobj['contextFeatures'] = [{'timeofday': timeofday, 'weather': weather, 'name': name}]
    return [name, weather, timeofday]
```


### <a name="add-all-coffee-data"></a>Incorporación de todos los datos de café

Esta función agrega la lista completa de café al objeto JSON que se va a enviar a la solicitud Rank.

La celda no tiene ninguna salida. Cuando se llama a la función, cambia el elemento `rankjsonobj`.


A continuación se muestra el ejemplo de las características de un solo café:

```json
{
    "id": "Cappucino",
    "features": [
    {
        "type": "hot",
        "origin": "kenya",
        "organic": "yes",
        "roast": "dark"

    }
}
```

```python
def add_action_features(rankjsonobj):
    rankjsonobj["actions"] = actionfeaturesobj
```

### <a name="compare-prediction-with-known-user-preference"></a>Comparación de la predicción con la preferencia de usuario conocida

Se llama a esta función después de llamar a la API Rank para cada iteración.

Esta función compara las preferencia de café del usuario, según la condición meteorológica y la hora del día, con la sugerencia de Personalizer para el usuario con esos filtros. Si la sugerencia coincide, se devuelve una puntuación de 1; de lo contrario, la puntuación es 0. La celda no tiene ninguna salida. Cuando se llama a la función, la salida indica la puntuación.

```python
def get_reward_from_simulated_data(name, weather, timeofday, prediction):
    if(userpref[name][weather][timeofday] == str(prediction)):
        return 1
    return 0
```

### <a name="loop-through-calls-to-rank-and-reward"></a>Bucle mediante llamadas a Rank y Reward

La siguiente celda es el _principal_ trabajo del cuaderno: obtener un usuario aleatorio, obtener la lista de cafés y enviar ambos a la API Rank. Después, comparar la predicción con las preferencias conocidas del usuario y enviar la recompensa de vuelta al servicio Personalizer.

El bucle se ejecuta las veces indicadas en `num_requests`. Personalizer necesita unos pocos miles de llamadas a Rank y Reward para crear un modelo.

A continuación se muestra un ejemplo de los datos JSON que se envían a la API Rank. Por brevedad, la lista de cafés no está completa. Puede ver el JSON completo de los cafés en `coffee.json`.

JSON enviado a la API Rank:

```json
{
   'contextFeatures':[
      {
         'timeofday':'Evening',
         'weather':'Snowy',
         'name':'Alice'
      }
   ],
   'actions':[
      {
         'id':'Cappucino',
         'features':[
            {
               'type':'hot',
               'origin':'kenya',
               'organic':'yes',
               'roast':'dark'
            }
         ]
      }
        ...rest of coffee list
   ],
   'excludedActions':[

   ],
   'eventId':'b5c4ef3e8c434f358382b04be8963f62',
   'deferActivation':False
}
```

Respuesta JSON de la API Rank:

```json
{
    'ranking': [
        {'id': 'Latte', 'probability': 0.85 },
        {'id': 'Iced mocha', 'probability': 0.05 },
        {'id': 'Cappucino', 'probability': 0.05 },
        {'id': 'Cold brew', 'probability': 0.05 }
    ],
    'eventId': '5001bcfe3bb542a1a238e6d18d57f2d2',
    'rewardActionId': 'Latte'
}
```

Por último, cada bucle muestra la selección aleatoria de usuario, condición meteorológica y hora del día, y determina la recompensa. La recompensa de 1 indica que el recurso Personalizer seleccionó el tipo de café correcto para el usuario, la condición meteorológica y la hora del día indicados.

```console
1 Alice Rainy Morning Latte 1
```

La función usa:

* Rank: una API REST POST para [obtener la clasificación](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/Rank).
* Reward: una API REST POST para [comunicar la recompensa](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/Reward).

```python
def iterations(n, modelCheck, jsonFormat):

    i = 1

    # default reward value - assumes failed prediction
    reward = 0

    # Print out dateTime
    currentDateTime()

    # collect results to aggregate in graph
    total = 0
    rewards = []
    count = []

    # default list of user, weather, time of day
    namesopt = ['Alice', 'Bob', 'Cathy', 'Dave']
    weatheropt = ['Sunny', 'Rainy', 'Snowy']
    timeofdayopt = ['Morning', 'Afternoon', 'Evening']


    while(i <= n):

        # create unique id to associate with an event
        eventid = add_event_id(jsonFormat)

        # generate a random sample
        [name, weather, timeofday] = add_random_user_and_contextfeatures(namesopt, weatheropt, timeofdayopt, jsonFormat)

        # add action features to rank
        add_action_features(jsonFormat)

        # show JSON to send to Rank
        print('To: ', jsonFormat)

        # choose an action - get prediction from Personalizer
        response = requests.post(personalization_rank_url, headers = headers, params = None, json = jsonFormat)

        # show Rank prediction
        print ('From: ',response.json())

        # compare personalization service recommendation with the simulated data to generate a reward value
        prediction = json.dumps(response.json()["rewardActionId"]).replace('"','')
        reward = get_reward_from_simulated_data(name, weather, timeofday, prediction)

        # show result for iteration
        print(f'   {i} {currentDateTime()} {name} {weather} {timeofday} {prediction} {reward}')

        # send the reward to the service
        response = requests.post(personalization_reward_url + eventid + "/reward", headers = headers, params= None, json = { "value" : reward })

        # for every N rank requests, compute total correct  total
         total =  total + reward

        # every N iteration, get last updated model date and time
        if(i % modelCheck == 0):

            print("**** 10% of loop found")

            get_last_updated(modelLastModified)

        # aggregate so chart is easier to read
        if(i % 10 == 0):
            rewards.append( total)
            count.append(i)
             total = 0

        i = i + 1

    # Print out dateTime
    currentDateTime()

    return [count, rewards]
```

## <a name="run-for-10000-iterations"></a>Ejecución durante 10 000 iteraciones
Ejecute el bucle de Personalizer durante 10 000 iteraciones. Se trata de un evento de larga duración. No cierre el explorador que ejecuta el cuaderno. Actualice el gráfico de métricas en Azure Portal periódicamente para ver el número total de llamadas al servicio. Cuando tenga en torno a 20 000 llamadas y una llamada a Rank y Reward para cada iteración del bucle, las iteraciones habrán terminado.

```python
# max iterations
num_requests = 200

# check last mod date N% of time - currently 10%
lastModCheck = int(num_requests * .10)

jsonTemplate = rankactionsjsonobj

# main iterations
[count, rewards] = iterations(num_requests, lastModCheck, jsonTemplate)
```



## <a name="chart-results-to-see-improvement"></a>Resultados del gráfico para ver la mejora

Cree un gráfico a partir de los valores de `count` y `rewards`.

```python
def createChart(x, y):
    plt.plot(x, y)
    plt.xlabel("Batch of rank events")
    plt.ylabel("Correct recommendations per batch")
    plt.show()
```

## <a name="run-chart-for-10000-rank-requests"></a>Ejecución del gráfico para 10 000 solicitudes Rank

Ejecute la función `createChart`.

```python
createChart(count,rewards)
```

## <a name="reading-the-chart"></a>Lectura del gráfico

Este gráfico muestra el éxito del modelo para la directiva de aprendizaje predeterminada actual.

![Este gráfico muestra el éxito de la directiva de aprendizaje actual para la duración de la prueba.](./media/tutorial-azure-notebook/azure-notebook-chart-results.png)


El objetivo ideal es que, al final de la prueba, el bucle ofrezca una tasa de éxito media próxima al 100 % menos la exploración. El valor predeterminado de exploración es 20 %.

`100-20=80`

Este valor de exploración se encuentra en la página **Configuración** del recurso Personalizer de Azure Portal.

Si quiere encontrar una directiva de aprendizaje mejor, en función de los datos de la API Rank, ejecute una [evaluación sin conexión](how-to-offline-evaluation.md) en el portal para el bucle de Personalizer.

## <a name="run-an-offline-evaluation"></a>Ejecución de una evaluación sin conexión

1. En Azure Portal, abra la página **Evaluaciones** del recurso Personalizer.
1. Seleccione **Crear evaluación**.
1. Escriba los datos necesarios del nombre de evaluación y el intervalo de fechas para la evaluación del bucle. El intervalo de fechas debe incluir solo los días en los que se centra para la evaluación.
    ![En Azure Portal, abra la página Evaluaciones del recurso Personalizer. Seleccione Crear evaluación. Escriba el nombre de la evaluación y el intervalo de fechas.](./media/tutorial-azure-notebook/create-offline-evaluation.png)

    La finalidad de la ejecución de esta evaluación sin conexión es determinar si hay una directiva de aprendizaje mejor para las características y las acciones que se usan en este bucle. Para encontrar esa directiva de aprendizaje mejor, asegúrese de que la opción **Detección de optimización** está activada.

1. Seleccione **Aceptar** para iniciar la evaluación.
1. En esta página **Evaluaciones** se muestra la nueva evaluación y su estado actual. En función de la cantidad de datos que tenga, puede tardar algún tiempo. Puede volver a esta página después de unos minutos para ver los resultados.
1. Una vez finalizada la evaluación, selecciónela y, a continuación, seleccione **Comparison de directivas de aprendizaje distintas**. Se muestran las directivas de aprendizaje disponibles y cómo se comportarían con los datos.
1. Seleccione la directiva de aprendizaje de nivel superior de la tabla y seleccione **Aplicar**. Esto aplica la _mejor_ directiva de aprendizaje al modelo y vuelve a entrenar.

## <a name="change-update-model-frequency-to-5-minutes"></a>Cambio de la frecuencia de actualización del modelo a 5 minutos

1. En Azure Portal, todavía en el recurso Personalizer, seleccione la página **Configuración**.
1. Cambie los valores de **Frecuencia de actualización del modelo** y **Tiempo de espera de recompensa** a 5 minutos y seleccione **Guardar**.

Más información sobre [Tiempo de espera de recompensa](concept-rewards.md#reward-wait-time) y [Frecuencia de actualización del modelo](how-to-settings.md#model-update-frequency).

```python
#Verify new learning policy and times
get_service_settings()
```

Compruebe que los valores de `rewardWaitTime` y `modelExportFrequency` de la salida están establecidos en 5 minutos.
```console
-----checking model
<Response [200]>
{'creationTime': '0001-01-01T00:00:00+00:00', 'lastModifiedTime': '0001-01-01T00:00:00+00:00'}
-----model updated: "0001-01-01T00:00:00+00:00"
-----checking service settings
<Response [200]>
{...learning policy...}
<Response [200]>
{'rewardWaitTime': '00:05:00', 'defaultReward': 0.0, 'rewardAggregation': 'earliest', 'explorationPercentage': 0.2, 'modelExportFrequency': '00:05:00', 'logRetentionDays': -1}
User count 4
Coffee count 4
```

## <a name="validate-new-learning-policy"></a>Comprobación de la nueva directiva de aprendizaje

Vuelva al archivo Azure Notebooks y continúe ejecutando el mismo bucle, pero solo durante 2000 iteraciones. Actualice el gráfico de métricas en Azure Portal periódicamente para ver el número total de llamadas al servicio. Cuando tenga en torno a 4000 llamadas y una llamada a Rank y Reward para cada iteración del bucle, las iteraciones habrán terminado.

```python
# max iterations
num_requests = 2000

# check last mod date N% of time - currently 10%
lastModCheck2 = int(num_requests * .10)

jsonTemplate2 = rankactionsjsonobj

# main iterations
[count2, rewards2] = iterations(num_requests, lastModCheck2, jsonTemplate)
```

## <a name="run-chart-for-2000-rank-requests"></a>Ejecución del gráfico de 2000 solicitudes Rank

Ejecute la función `createChart`.

```python
createChart(count2,rewards2)
```

## <a name="review-the-second-chart"></a>Revisión del segundo gráfico

El segundo gráfico debe mostrar un aumento visible en las predicciones de Rank, en correspondencia con las preferencias del usuario.

![El segundo gráfico debe mostrar un aumento visible en las predicciones de Rank, en correspondencia con las preferencias del usuario.](./media/tutorial-azure-notebook/azure-notebook-chart-results-happy-graph.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no desea continuar con la serie de tutoriales, limpie los siguientes recursos:

* Elimine el proyecto de Azure Notebook.
* Elimine el recurso Personalizer.

## <a name="next-steps"></a>Pasos siguientes

El [cuaderno de Jupyter Notebook y los archivos de datos](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/tree/master/samples/azurenotebook) usados en este ejemplo están disponibles en el repositorio de GitHub para Personalizer.

