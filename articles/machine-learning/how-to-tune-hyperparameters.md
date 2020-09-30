---
title: Ajuste de los hiperparámetros de un modelo
titleSuffix: Azure Machine Learning
description: Ajuste de forma eficaz los hiperparámetros de su modelo de aprendizaje profundo o aprendizaje automático mediante Azure Machine Learning. Aprenderá cómo definir el espacio de búsqueda de parámetros, especificar una métrica principal para optimizar y terminar anticipadamente las series de bajo rendimiento.
ms.author: swatig
author: swatig007
ms.reviewer: sgilley
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.date: 03/30/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 04942c745548903a5f8092bc5b04ea2152029726
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90885925"
---
# <a name="tune-hyperparameters-for-your-model-with-azure-machine-learning"></a>Ajuste de los hiperparámetros de un modelo mediante Azure Machine Learning


Ajuste de forma eficaz los hiperparámetros de un modelo mediante Azure Machine Learning.  El ajuste de hiperparámetros incluye los siguientes pasos:

* Definir el espacio de búsqueda de parámetros
* Especificar una métrica principal para optimizar  
* Especificar criterios de finalización anticipada para series de bajo rendimiento
* Asignar recursos para el ajuste de hiperparámetros
* Iniciar un experimento con la configuración anterior
* Visualizar las series de entrenamiento
* Seleccionar la configuración de rendimiento óptima para el modelo

## <a name="what-are-hyperparameters"></a>¿Qué son los hiperparámetros?

Los hiperparámetros son parámetros ajustables que se eligen para entrenar un modelo y que rigen el propio proceso de entrenamiento. Por ejemplo, para entrenar una red neuronal profunda, debe decidir el número de capas ocultas en la red y la cantidad de nodos de cada capa antes de entrenar al modelo. Estos valores suelen permanecer constantes durante el proceso de entrenamiento.

En escenarios de aprendizaje profundo o aprendizaje automático, el rendimiento del modelo depende en gran medida de los valores de hiperparámetro seleccionados. El objetivo de la exploración de los hiperparámetros es buscar entre diversas configuraciones de hiperparámetros hasta dar con la que tenga como resultado un rendimiento óptimo. Normalmente, el proceso de exploración de hiperparámetros es un trabajo manual muy laborioso, dado que el espacio de búsqueda es muy extenso y la evaluación de cada configuración puede ser costosa.

Azure Machine Learning le permite automatizar la exploración de hiperparámetros de manera eficiente, de forma que le ahorra una cantidad considerable de tiempo y recursos. Especifique el intervalo de valores de hiperparámetro y un número máximo de series de entrenamiento. El sistema inicia entonces automáticamente varias series simultáneas con configuraciones de parámetros diferentes y busca la configuración que da como resultado un rendimiento óptimo, según se mida mediante la métrica que haya elegido. Las series de entrenamientos con bajo rendimiento se terminan automáticamente de forma anticipada, lo que reduce el desperdicio de recursos de proceso. En su lugar, estos recursos se usan para explorar otras configuraciones de hiperparámetros.


## <a name="define-search-space"></a>Definición del espacio de búsqueda

Ajuste automáticamente los hiperparámetros explorando el intervalo de valores definidos para cada hiperparámetro.

### <a name="types-of-hyperparameters"></a>Tipos de hiperparámetros

Cada hiperparámetro puede ser discreto o continuo y tiene una distribución de valores que se describe con una [expresión de parámetro](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.hyperdrive.parameter_expressions?view=azure-ml-py&preserve-view=true).

#### <a name="discrete-hyperparameters"></a>Hiperparámetros discretos 

Los hiperparámetros discretos se especifican con un objeto `choice` entre valores discretos. `choice` puede ser:

* uno o más valores separados por comas;
* un objeto `range`;
* cualquier objeto `list` arbitrario.


```Python
    {
        "batch_size": choice(16, 32, 64, 128)
        "number_of_hidden_layers": choice(range(1,5))
    }
```

En este caso, `batch_size` toma uno de los valores [16, 32, 64, 128] y `number_of_hidden_layers` toma uno de los valores [1, 2, 3, 4].

También se pueden especificar hiperparámetros discretos avanzados mediante una distribución. Se admiten las siguientes distribuciones:

* `quniform(low, high, q)`: devuelve un valor como round(uniform(low, high) / q) * q
* `qloguniform(low, high, q)`: devuelve un valor como round(exp(uniform(low, high)) / q) * q
* `qnormal(mu, sigma, q)`: devuelve un valor como round(normal(mu, sigma) / q) * q
* `qlognormal(mu, sigma, q)`: devuelve un valor como round(exp(normal(mu, sigma)) / q) * q

#### <a name="continuous-hyperparameters"></a>Hiperparámetros continuos 

Los hiperparámetros continuos se especifican como una distribución a través de un intervalo continuo de valores. Las distribuciones admitidas son:

* `uniform(low, high)`: devuelve un valor distribuido uniformemente entre bajo y alto.
* `loguniform(low, high)`: devuelve un valor que se extrae según exp(uniform(low, high)) de forma que el logaritmo del valor devuelto se distribuye uniformemente.
* `normal(mu, sigma)`: devuelve un valor real que se distribuye normalmente con media mu y desviación estándar sigma.
* `lognormal(mu, sigma)`: devuelve un valor extraído según exp(normal(mu, sigma)) de forma que el logaritmo del valor devuelto se distribuye normalmente.

El siguiente es un ejemplo de definición de espacio de parámetros:

```Python
    {    
        "learning_rate": normal(10, 3),
        "keep_probability": uniform(0.05, 0.1)
    }
```

Este código define un espacio de búsqueda con dos parámetros: `learning_rate` y `keep_probability`. `learning_rate` tiene una distribución normal con un valor medio de 10 y una desviación estándar de 3. `keep_probability` tiene una distribución uniforme con un valor mínimo de 0,05 y un valor máximo de 0,1.

### <a name="sampling-the-hyperparameter-space"></a>Muestreo del espacio de hiperparámetros

También puede especificar el método de muestreo de parámetros que se usará durante la definición del espacio de hiperparámetros. Azure Machine Learning admite muestreo aleatorio, muestreo de cuadrícula y muestreo bayesiano.

#### <a name="picking-a-sampling-method"></a>Selección de un método de muestreo

* El muestreo de cuadrícula se puede usar si el espacio de hiperparámetros se puede definir como una opción entre valores discretos y si tiene el presupuesto suficiente para buscar de forma exhaustiva todos los valores en el espacio de búsqueda definido. Además, se puede usar la finalización anticipada automatizada de las ejecuciones de bajo rendimiento, lo que reduce el desperdicio de los recursos.
* El muestreo aleatorio permite que el espacio de hiperparámetros incluya tanto hiperparámetros discretos como continuos. En la práctica, se generan buenos resultados la mayoría de las veces y también se permite el uso de la finalización anticipada automatizada de las ejecuciones de bajo rendimiento. Algunos usuarios realizan una búsqueda inicial mediante el muestreo aleatorio y luego refinan el espacio de búsqueda de forma iterativa para mejorar los resultados.
* El muestreo bayesiano aprovecha el conocimiento de las muestras anteriores al elegir los valores de hiperparámetro, intentando mejorar de forma eficaz la métrica principal indicada. El muestreo bayesiano se recomienda cuando se tiene el presupuesto suficiente para explorar el espacio de hiperparámetros. Para obtener los mejores resultados con el muestreo bayesiano, se recomienda usar un número máximo de ejecuciones mayor o igual que veinte veces el número de hiperparámetros que se está optimizando. Tenga en cuenta que el muestreo bayesiano no admite actualmente ninguna directiva de terminación anticipada.

#### <a name="random-sampling"></a>Muestreo aleatorio

En el muestreo aleatorio, los valores de hiperparámetro se seleccionan aleatoriamente del espacio de búsqueda definido. El [muestreo aleatorio](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.hyperdrive.randomparametersampling?view=azure-ml-py&preserve-view=true) permite que el espacio de búsqueda incluya hiperparámetros discretos y continuos.

```Python
from azureml.train.hyperdrive import RandomParameterSampling
from azureml.train.hyperdrive import normal, uniform, choice
param_sampling = RandomParameterSampling( {
        "learning_rate": normal(10, 3),
        "keep_probability": uniform(0.05, 0.1),
        "batch_size": choice(16, 32, 64, 128)
    }
)
```

#### <a name="grid-sampling"></a>Muestreo de cuadrícula

El [muestreo de cuadrícula](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.hyperdrive.gridparametersampling?view=azure-ml-py&preserve-view=true) realiza una búsqueda de cuadrícula sencilla por todos los valores posibles del espacio de búsqueda definido. Solo se puede usar con hiperparámetros especificados con `choice`. Por ejemplo, el siguiente espacio tiene seis muestras en total:

```Python
from azureml.train.hyperdrive import GridParameterSampling
from azureml.train.hyperdrive import choice
param_sampling = GridParameterSampling( {
        "num_hidden_layers": choice(1, 2, 3),
        "batch_size": choice(16, 32)
    }
)
```

#### <a name="bayesian-sampling"></a>Muestreo bayesiano

El [muestreo bayesiano](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.hyperdrive.bayesianparametersampling?view=azure-ml-py&preserve-view=true) se basa en el algoritmo de optimización bayesiano y toma decisiones inteligentes sobre los valores de hiperparámetro que se van a muestrear a continuación. La muestra se toma en función de cómo se hayan ejecutado las muestras anteriores, de tal forma que la nueva muestra mejora la métrica principal notificada.

Cuando se usa el muestreo bayesiano, el número de ejecuciones simultáneas tiene un efecto sobre la eficacia del proceso de ajuste. Normalmente, un menor número de ejecuciones simultáneas puede provocar una mejor convergencia de muestreo, dado que el menor grado de paralelismo aumenta el número de ejecuciones que se benefician de las ejecuciones completadas previamente.

El muestreo bayesiano solo admite las distribuciones `choice`, `uniform` y `quniform` en el espacio de búsqueda.

```Python
from azureml.train.hyperdrive import BayesianParameterSampling
from azureml.train.hyperdrive import uniform, choice
param_sampling = BayesianParameterSampling( {
        "learning_rate": uniform(0.05, 0.1),
        "batch_size": choice(16, 32, 64, 128)
    }
)
```

> [!NOTE]
> El muestreo bayesiano no admite ninguna directiva de terminación anticipada (consulte [especificación de una directiva de terminación anticipada](#early-termination)). Al utilizar el muestreo de parámetros bayesiano, establezca el valor en `early_termination_policy = None`, o no añada el parámetro `early_termination_policy`.

## <a name="specify-primary-metric"></a><a name="specify-primary-metric-to-optimize"></a> Especificación de la métrica principal

Especifique la [métrica principal](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.hyperdrive.primarymetricgoal?view=azure-ml-py&preserve-view=true) que quiere que se optimice con el experimento de ajuste de hiperparámetros. En cada serie de entrenamiento se evalúa la métrica principal. Las ejecuciones con un rendimiento deficiente (donde la métrica principal no satisface los criterios establecidos por la directiva de terminación anticipada) se terminan. Además del nombre de la métrica principal, también especificará el objetivo de la optimización: si maximizar o minimizar la métrica principal.

* `primary_metric_name`: nombre de la métrica principal a optimizar. El nombre de la métrica principal debe coincidir exactamente con el nombre de la métrica registrado por el script de entrenamiento. Consulte [Registrar métricas para el ajuste de hiperparámetros](#log-metrics-for-hyperparameter-tuning).
* `primary_metric_goal`: puede ser `PrimaryMetricGoal.MAXIMIZE` o `PrimaryMetricGoal.MINIMIZE` y determina si la métrica principal se maximizará o minimizará al evaluar las ejecuciones. 

```Python
primary_metric_name="accuracy",
primary_metric_goal=PrimaryMetricGoal.MAXIMIZE
```

Optimice las ejecuciones para maximizar la "precisión".  Asegúrese de registrar este valor en el script de entrenamiento.

### <a name="specify-primary-metric"></a><a name="log-metrics-for-hyperparameter-tuning"></a> Especificación de la métrica principal

El script de entrenamiento para el modelo debe registrar las métricas pertinentes durante el entrenamiento del modelo. Cuando se configura el ajuste de hiperparámetros, debe especificar la métrica principal que se utilizará para evaluar el rendimiento de las ejecuciones. (Consulte [especificación de una métrica principal para optimizar](#specify-primary-metric-to-optimize)).  En el script de entrenamiento, debe registrar esta métrica de modo que esté disponible para el proceso de ajuste de hiperparámetros.

Registre la métrica del script de entrenamiento mediante el siguiente fragmento de código de ejemplo:

```Python
from azureml.core.run import Run
run_logger = Run.get_context()
run_logger.log("accuracy", float(val_accuracy))
```

El script de entrenamiento calcula el valor `val_accuracy` y lo registra como "precisión", que se usa como métrica principal. Cada vez que se registra la métrica, el servicio de ajuste de hiperparámetros la recibe. Es el desarrollador del modelo quien debe determinar la frecuencia con que se notifica esta métrica.

## <a name="specify-early-termination-policy"></a><a name="early-termination"></a> Especificación de una directiva de terminación anticipada

Finalice de forma automática las ejecuciones con un bajo rendimiento con la ayuda de una directiva de finalización anticipada. La finalización reduce el desperdicio de recursos, los cuales se usan para explorar otras configuraciones de parámetros.

Cuando se usa una directiva de terminación anticipada, puede configurar los siguientes parámetros que controlan cuándo se aplica una directiva:

* `evaluation_interval`: la frecuencia con que se aplica la directiva. Cada vez que el script de entrenamiento registra la métrica principal se considera un intervalo. Por lo tanto, un parámetro `evaluation_interval` de 1 aplicará la directiva cada vez que el script de entrenamiento notifique la métrica principal. Un parámetro `evaluation_interval` de 2 aplicará la directiva las demás veces que el script de entrenamiento notifique la métrica principal. Si no se especifica, `evaluation_interval` está establecido como 1 de forma predeterminada.
* `delay_evaluation`: retrasa la primera evaluación de la directiva un número especificado de intervalos. Es un parámetro opcional que permite que todas las configuraciones se ejecuten durante un número mínimo inicial de intervalos, lo que evita la terminación prematura de series de entrenamientos. Si se especifica, la directiva aplica cada múltiplo de evaluation_interval que sea mayor o igual que delay_evaluation.

Azure Machine Learning admite las siguientes directivas de terminación anticipada.

### <a name="bandit-policy"></a>Directiva de bandidos

La de [bandidos](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.hyperdrive.banditpolicy?view=azure-ml-py#&preserve-view=truedefinition) es una directiva de terminación basada en el factor de demora o la cantidad de demora y el intervalo de evaluación. La directiva termina anticipadamente aquellas ejecuciones en las que la métrica principal no se encuentre dentro del factor de demora o cantidad de demora especificadas con respecto a la serie de entrenamientos con el mejor rendimiento. Esta directiva toma los siguientes parámetros de configuración:

* `slack_factor` o `slack_amount`: la demora permitida con respecto a la serie de entrenamientos con el mejor rendimiento. `slack_factor` especifica la demora permitida como una relación. `slack_amount` especifica la demora permitida como una cantidad absoluta, en lugar de una relación.

    Por ejemplo, imagine que se aplica una directiva de bandidos en el intervalo 10. Suponga que la ejecución con el mejor rendimiento en el intervalo 10 notifica una métrica principal de 0,8 con el objetivo de maximizar esta. Si la directiva se especificó con un parámetro `slack_factor` de 0,2, se terminarán aquellas series de entrenamientos cuya mejor métrica en el intervalo 10 sea inferior a 0,66 (0,8/(1 +`slack_factor`)). Si, por el contrario, la directiva se especificó con un parámetro `slack_amount` de 0,2, se terminarán aquellas series de entrenamiento cuya mejor métrica en el intervalo 10 sea inferior a 0,6 (0,8 - `slack_amount`).
* `evaluation_interval`: la frecuencia con que se aplica la directiva (parámetro opcional).
* `delay_evaluation`: retrasa la primera evaluación de directiva un número especificado de intervalos (parámetro opcional).


```Python
from azureml.train.hyperdrive import BanditPolicy
early_termination_policy = BanditPolicy(slack_factor = 0.1, evaluation_interval=1, delay_evaluation=5)
```

En este ejemplo, se aplica la directiva de terminación anticipada en cada intervalo cuando se notifican las métricas, comenzando en el intervalo de evaluación 5. Cualquier ejecución cuya mejor métrica sea inferior a (1/(1+0,1), o al 91 % de la ejecución con el mejor rendimiento, se terminará.

### <a name="median-stopping-policy"></a>Directiva de mediana de detención

La [mediana de detención](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.hyperdrive.medianstoppingpolicy?view=azure-ml-py&preserve-view=true) es una directiva de terminación anticipada basada en la ejecución de valores medios de las métricas principales notificadas por las ejecuciones. Esta directiva calcula los valores medios de ejecución en todas las series de entrenamientos y termina las ejecuciones cuyo rendimiento es peor que la mediana de los valores medios de ejecución. Esta directiva toma los parámetros de configuración siguientes:
* `evaluation_interval`: la frecuencia con que se aplica la directiva (parámetro opcional).
* `delay_evaluation`: retrasa la primera evaluación de directiva un número especificado de intervalos (parámetro opcional).


```Python
from azureml.train.hyperdrive import MedianStoppingPolicy
early_termination_policy = MedianStoppingPolicy(evaluation_interval=1, delay_evaluation=5)
```

En este ejemplo, se aplica la directiva de terminación anticipada en cada intervalo, comenzando en el intervalo de evaluación 5. Una ejecución se termina en el intervalo 5 si su mejor métrica principal es peor que la mediana de los valores medios de ejecución durante los intervalos en una relación de 1 a 5 en todas las series de entrenamientos.

### <a name="truncation-selection-policy"></a>Directiva de selección de truncamiento

La [selección de truncamiento](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.hyperdrive.truncationselectionpolicy?view=azure-ml-py&preserve-view=true) cancela un porcentaje dado de ejecuciones con el rendimiento más bajo en cada intervalo de evaluación. Las ejecuciones se comparan en función de su rendimiento en la métrica principal y las X % con el rendimiento más bajo se terminan. Esta directiva toma los siguientes parámetros de configuración:

* `truncation_percentage`: el porcentaje de ejecuciones con el rendimiento más bajo que se terminarán en cada intervalo de evaluación. Especifique un valor entero comprendido entre 1 y 99.
* `evaluation_interval`: la frecuencia con que se aplica la directiva (parámetro opcional).
* `delay_evaluation`: retrasa la primera evaluación de directiva un número especificado de intervalos (parámetro opcional).


```Python
from azureml.train.hyperdrive import TruncationSelectionPolicy
early_termination_policy = TruncationSelectionPolicy(evaluation_interval=1, truncation_percentage=20, delay_evaluation=5)
```

En este ejemplo, se aplica la directiva de terminación anticipada en cada intervalo, comenzando en el intervalo de evaluación 5. Una ejecución se terminará en el intervalo 5 si su rendimiento en este intervalo se encuentra en el 20 % del rendimiento más bajo de todas las ejecuciones en el intervalo 5.

### <a name="no-termination-policy"></a>Sin directiva de terminación

Si quiere que todas las series de entrenamientos se ejecuten hasta su finalización, establezca la directiva en None. El efecto será el de no aplicar ninguna directiva de terminación anticipada.

```Python
policy=None
```

### <a name="default-policy"></a>Directiva predeterminada

Si no se especifica ninguna directiva, el servicio de ajuste de hiperparámetros permitirá que todas las series de entrenamiento se ejecuten hasta completarse.

### <a name="picking-an-early-termination-policy"></a>Selección de una directiva de terminación anticipada

* Si está buscando una directiva conservadora que proporcione ahorros sin finalizar trabajos prometedores, puede usar una Directiva de mediana de detención con `evaluation_interval` 1 y `delay_evaluation` 5. Se trata de una configuración conservadora que puede proporcionar unos ahorros de entre un 25 % y un 35 % sin pérdidas de la métrica principal (según nuestros datos de evaluación).
* Si busca un ahorro más agresivo a partir de la terminación anticipada, puede usar la directiva de bandidos con una directiva de selección de truncamiento o demora permisible más estricta (menor) con un porcentaje de truncamiento mayor.

## <a name="allocate-resources"></a>Asignación de recursos

Controle el presupuesto de recursos para el experimento de ajuste de hiperparámetros especificando el número total máximo de ejecuciones de entrenamiento.  Opcionalmente, especifique la duración máxima para el experimento de ajuste de hiperparámetros.

* `max_total_runs`: número total máximo de ejecuciones de entrenamiento que se crearán. Límite superior: por ejemplo, se pueden tener menos ejecuciones si el espacio de hiperparámetros es finito y tiene menos muestras. Debe ser un número entre 1 y 1000.
* `max_duration_minutes`: duración máxima en minutos del experimento de ajuste de hiperparámetros. El parámetro es opcional y, si existe, automáticamente se cancelan todas las series que podrían estar ejecutándose después de esta duración.

>[!NOTE] 
>Si se especifica `max_total_runs` y `max_duration_minutes`, el experimento de ajuste de hiperparámetros finaliza cuando se alcanza el primero de estos dos umbrales.

Además, especifique el número máximo de series de entrenamientos que se ejecutarán al mismo tiempo durante la búsqueda de ajuste de hiperparámetros.

* `max_concurrent_runs`: número máximo de series que se ejecutarán al mismo tiempo en un momento dado. Si no especifica ninguno, el número de ejecuciones de `max_total_runs` se iniciarán en paralelo. Si se especifica, el valor debe ser un número entre 1 y 100.

>[!NOTE] 
>El número de ejecuciones simultáneas viene determinado por los recursos disponibles en el destino de proceso especificado. Por lo tanto, debe asegurarse de que el destino de proceso tenga los recursos disponibles para la simultaneidad deseada.

Asigne recursos para el ajuste de hiperparámetros:

```Python
max_total_runs=20,
max_concurrent_runs=4
```

En este código se configura un experimento de ajuste de hiperparámetros para usar un máximo de 20 ejecuciones totales, de forma tal que se ejecutan cuatro configuraciones al mismo tiempo.

## <a name="configure-experiment"></a>Configuración del experimento

[Configure el experimento de ajuste de hiperparámetros](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.hyperdrive.hyperdriverunconfig?view=azure-ml-py&preserve-view=true) mediante el espacio de búsqueda de hiperparámetros definido, las directivas de terminación anticipada, las métricas principales y la asignación de recursos que ya se han visto en las secciones anteriores. Además, proporcione un parámetro `estimator` que se invocará con los hiperparámetros muestreados. El parámetro `estimator` describe el script de entrenamiento que se ejecutará, los recursos por trabajo (una o varias GPU) y el destino de proceso que se usará. Dado que la simultaneidad en el experimento de ajuste de hiperparámetros viene determinada por los recursos disponibles, asegúrese de que el destino de proceso especificado en `estimator` tenga recursos suficientes para la simultaneidad deseada. (Para obtener más información sobre los estimadores, consulte [cómo entrenar modelos](how-to-train-ml-models.md)).

Configure el experimento de ajuste de hiperparámetros:

```Python
from azureml.train.hyperdrive import HyperDriveConfig
hyperdrive_run_config = HyperDriveConfig(estimator=estimator,
                          hyperparameter_sampling=param_sampling, 
                          policy=early_termination_policy,
                          primary_metric_name="accuracy", 
                          primary_metric_goal=PrimaryMetricGoal.MAXIMIZE,
                          max_total_runs=100,
                          max_concurrent_runs=4)
```

## <a name="submit-experiment"></a>Envío del experimento

Tras definir la configuración de ajuste de hiperparámetros, [envíe un experimento](https://docs.microsoft.com/python/api/azureml-core/azureml.core.experiment%28class%29?view=azure-ml-py#&preserve-view=truesubmit-config--tags-none----kwargs-):

```Python
from azureml.core.experiment import Experiment
experiment = Experiment(workspace, experiment_name)
hyperdrive_run = experiment.submit(hyperdrive_run_config)
```

`experiment_name` es el nombre que quiere asignar a su experimento de ajuste de hiperparámetros y `workspace` es el área de trabajo en la que quiere crear el experimento (para más información sobre los experimentos, consulte [¿Cómo funciona Azure Machine Learning?](concept-azure-machine-learning-architecture.md)).

## <a name="warm-start-your-hyperparameter-tuning-experiment-optional"></a>Inicio en caliente del experimento de ajuste de hiperparámetros (opcional)

A menudo, buscar los mejores valores de hiperparámetros para el modelo puede ser un proceso iterativo, que necesita varias ejecuciones de optimización que aprenden de las ejecuciones de optimización de hiperparámetros anteriores. La reutilización del conocimiento de estas ejecuciones anteriores acelerará el proceso de ajuste de hiperparámetros, con lo que se reduce el costo de optimizar el modelo y se mejora potencialmente la métrica principal del modelo resultante. Cuando se inicia en caliente un experimento de optimización de hiperparámetros con el muestreo bayesiano, se usarán las pruebas de la ejecución anterior como conocimiento previo para elegir de forma inteligente nuevas muestras, con el fin de mejorar la métrica principal. Además, cuando se usa el muestreo aleatorio o de cuadrícula, cualquier decisión de terminación anticipada aprovechará las métricas de las ejecuciones anteriores para determinar las ejecuciones de entrenamiento de bajo rendimiento. 

Azure Machine Learning permite iniciar en caliente la ejecución de optimización de hiperparámetros aprovechando el conocimiento de hasta cinco ejecuciones primarias de optimización de hiperparámetros previamente completadas o canceladas. Puede especificar la lista de ejecuciones principales que desea iniciar en caliente utilizando este fragmento de código:

```Python
from azureml.train.hyperdrive import HyperDriveRun

warmstart_parent_1 = HyperDriveRun(experiment, "warmstart_parent_run_ID_1")
warmstart_parent_2 = HyperDriveRun(experiment, "warmstart_parent_run_ID_2")
warmstart_parents_to_resume_from = [warmstart_parent_1, warmstart_parent_2]
```

Además, puede haber ocasiones en las que se cancelan las ejecuciones de entrenamiento individuales de un experimento de optimización de hiperparámetros a causa de limitaciones de presupuesto o errores debidos a otras razones. Ahora es posible reanudar estas ejecuciones de entrenamiento individuales a partir del último punto de control (suponiendo que el script de entrenamiento controla los puntos de control). Al reanudar una ejecución de entrenamiento individual, se usará la misma configuración de hiperparámetros y se montará la carpeta de salidas que se usa para esa ejecución. El script de entrenamiento debe aceptar el argumento `resume-from`, que contiene los archivos de punto de control o de modelo a partir de los que se reanuda la ejecución de entrenamiento. Puede reanudar las ejecuciones de entrenamiento individuales mediante el siguiente fragmento de código:

```Python
from azureml.core.run import Run

resume_child_run_1 = Run(experiment, "resume_child_run_ID_1")
resume_child_run_2 = Run(experiment, "resume_child_run_ID_2")
child_runs_to_resume = [resume_child_run_1, resume_child_run_2]
```

Puede configurar el experimento de optimización de hiperparámetros para que se inicie en caliente a partir de un experimento anterior o para reanudar las ejecuciones de entrenamiento individuales usando los parámetros opcionales `resume_from` y `resume_child_runs` de la configuración:

```Python
from azureml.train.hyperdrive import HyperDriveConfig

hyperdrive_run_config = HyperDriveConfig(estimator=estimator,
                          hyperparameter_sampling=param_sampling, 
                          policy=early_termination_policy,
                          resume_from=warmstart_parents_to_resume_from, 
                          resume_child_runs=child_runs_to_resume,
                          primary_metric_name="accuracy", 
                          primary_metric_goal=PrimaryMetricGoal.MAXIMIZE,
                          max_total_runs=100,
                          max_concurrent_runs=4)
```

## <a name="visualize-experiment"></a>Visualización del experimento

El SDK de Azure Machine Learning proporciona un [widget de Notebook](https://docs.microsoft.com/python/api/azureml-widgets/azureml.widgets.rundetails?view=azure-ml-py&preserve-view=true) que visualiza el progreso de las series de entrenamientos. El siguiente fragmento de código visualiza todas las ejecuciones de ajuste de hiperparámetros en un solo lugar, un Jupyter Notebook:

```Python
from azureml.widgets import RunDetails
RunDetails(hyperdrive_run).show()
```

En este código se muestra una tabla con detalles sobre las series de entrenamientos de cada una de las configuraciones de hiperparámetros.

![tabla de ajuste de hiperparámetros](./media/how-to-tune-hyperparameters/HyperparameterTuningTable.png)

También puede visualizar el rendimiento de cada una de las ejecuciones a medida que progresa el entrenamiento. 

![trazado de ajuste de hiperparámetros](./media/how-to-tune-hyperparameters/HyperparameterTuningPlot.png)

Además, puede identificar visualmente la correlación entre el rendimiento y los valores de hiperparámetros individuales mediante un trazado de coordenadas paralelas. 

[![coordenadas paralelas de ajuste de hiperparámetros](./media/how-to-tune-hyperparameters/HyperparameterTuningParallelCoordinates.png)](media/how-to-tune-hyperparameters/hyperparameter-tuning-parallel-coordinates-expanded.png)

Puede visualizar todas las ejecuciones de ajuste de hiperparámetros en el portal web de Azure. Para más información sobre cómo ver un experimento en el portal web, vea [cómo realizar el seguimiento de experimentos](how-to-monitor-view-training-logs.md#view-the-experiment-in-the-web-portal).

## <a name="find-the-best-model"></a>Identificación del mejor modelo

Una vez que se han completado todas las series de ajuste de hiperparámetros, [identifique la configuración con el mejor rendimiento](/python/api/azureml-train-core/azureml.train.hyperdrive.hyperdriverun?view=azure-ml-py#&preserve-view=trueget-best-run-by-primary-metric-include-failed-true--include-canceled-true--include-resume-from-runs-true-----typing-union-azureml-core-run-run--nonetype-) y los valores de hiperparámetro correspondientes:

```Python
best_run = hyperdrive_run.get_best_run_by_primary_metric()
best_run_metrics = best_run.get_metrics()
parameter_values = best_run.get_details()['runDefinition']['Arguments']

print('Best Run Id: ', best_run.id)
print('\n Accuracy:', best_run_metrics['accuracy'])
print('\n learning rate:',parameter_values[3])
print('\n keep probability:',parameter_values[5])
print('\n batch size:',parameter_values[7])
```

## <a name="sample-notebook"></a>Cuaderno de ejemplo
Consulte los cuadernos train-hyperparameter-* en esta carpeta:
* [how-to-use-azureml/training-with-deep-learning](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/training-with-deep-learning)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Pasos siguientes
* [Seguimiento de experimentos](how-to-track-experiments.md)
* [Implementar un modelo entrenado](how-to-deploy-and-where.md)
