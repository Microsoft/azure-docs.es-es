---
title: Integración de Azure Stream Analytics con Azure Machine Learning
description: En este artículo se explica cómo integrar un trabajo de Azure Stream Analytics con modelos de Azure Machine Learning.
author: sidram
ms.author: sidram
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 03/19/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: e2277e2088a8cb386d6f19799b235d96e08959b0
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543442"
---
# <a name="integrate-azure-stream-analytics-with-azure-machine-learning-preview"></a>Integración de Azure Stream Analytics con Azure Machine Learning (versión preliminar)

Puede implementar modelos de Machine Learning como funciones definidas por el usuario en los trabajos de Azure Stream Analytics para realizar predicciones y puntuaciones en tiempo real a partir de los datos de entrada de streaming. [Azure Machine Learning](../machine-learning/overview-what-is-azure-ml.md) le permite usar cualquier herramienta de código abierto popular (como Tensorflow, scikit-learn o PyTorch) para preparar, entrenar e implementar modelos.

## <a name="prerequisites"></a>Requisitos previos

Siga los pasos que se indican a continuación antes de agregar un modelo de Machine Learning como función a su trabajo de Stream Analytics:

1. Use Azure Machine Learning para [implementar el modelo como servicio web](https://docs.microsoft.com/azure/machine-learning/how-to-deploy-and-where).

2. El script de puntuación debe tener [entradas y salidas de ejemplo](../machine-learning/how-to-deploy-and-where.md) que Azure Machine Learning usará para generar especificaciones de esquema. Stream Analytics usa el esquema para comprender la signatura de función del servicio web. Puede usar este [ejemplo de definición de Swagger](https://github.com/Azure/azure-stream-analytics/blob/master/Samples/AzureML/swagger-example.json) como referencia para asegurarse de que la ha configurado correctamente.

3. Asegúrese de que el servicio web acepta y devuelve datos serializados de JSON.

4. Implemente el modelo en [Azure Kubernetes Service](../machine-learning/how-to-deploy-and-where.md#choose-a-compute-target) para implementaciones de producción a gran escala. Si el servicio web no es capaz de controlar el número de solicitudes procedentes del trabajo, se reducirá el rendimiento del trabajo de Stream Analytics, lo que afectará a la latencia. Los modelos implementados en Azure Container Instances solo se admiten cuando se usa Azure Portal. Todavía no se admiten los modelos creados con [diseñador de Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/concept-designer) en Stream Analytics.

## <a name="add-a-machine-learning-model-to-your-job"></a>Incorporación de un modelo de Machine Learning en el trabajo

Puede agregar funciones de Azure Machine Learning a su trabajo de Stream Analytics directamente desde Azure Portal.

1. Vaya al trabajo de Stream Analytics en Azure Portal y seleccione **Funciones** en **Topología de trabajo**. A continuación, seleccione **Azure ML Service** en el menú desplegable **+ Agregar**.

   ![Agregar funciones definidas por el usuario de Azure Machine Learning](./media/machine-learning-udf/add-azureml-udf.png)

2. Rellene el formulario **Función de Azure Machine Learning Service** con los siguientes valores de propiedad:

   ![Configuración de función definida por el usuario de Azure Machine Learning](./media/machine-learning-udf/configure-azureml-udf.png)

En la tabla siguiente se describe cada una de las propiedades de las funciones de Azure Machine Learning Service en Stream Analytics.

|Propiedad|Descripción|
|--------|-----------|
|Alias de función|Escriba un nombre para invocar la función en la consulta.|
|Suscripción|Su suscripción de Azure.|
|Área de trabajo de Azure Machine Learning|Área de trabajo de Azure Machine Learning que usó para implementar el modelo como servicio web.|
|Implementaciones|Servicio web que hospeda el modelo.|
|Signatura de función|Firma del servicio web que se deduce a partir de la especificación del esquema de la API. Si no se puede cargar la firma, compruebe que ha proporcionado entradas y salidas de ejemplo en el script de puntuación para generar el esquema automáticamente.|
|Número de solicitudes paralelas por partición|Se trata de una configuración avanzada para optimizar el rendimiento a gran escala. Este número representa las solicitudes simultáneas enviadas desde cada partición del trabajo al servicio web. Los trabajos con seis unidades de streaming (SU) o menos tienen una partición. Los trabajos con 12 SU tienen dos particiones, los de 18 SU tienen tres particiones y así sucesivamente.<br><br> Por ejemplo, si el trabajo tiene dos particiones y establece este parámetro en cuatro, habrá ocho solicitudes simultáneas del trabajo al servicio web. En este momento de la versión preliminar pública, el valor predeterminado es 20 y no se puede actualizar.|
|Número máximo de lotes|Se trata de una configuración avanzada para la optimización del rendimiento a gran escala. Este número representa el número máximo de eventos que se van a procesar por lotes en una única solicitud enviada al servicio web.|

## <a name="supported-input-parameters"></a>Parámetros de entrada compatibles

Cuando la consulta de Stream Analytics invoca una función definida por el usuario de Azure Machine Learning, el trabajo crea una solicitud serializada de JSON para el servicio web. La solicitud se basa en un esquema específico del modelo. Debe proporcionar un ejemplo de entrada y salida en el script de puntuación para [generar un esquema automáticamente](../machine-learning/how-to-deploy-and-where.md). El esquema permite que Stream Analytics genere la solicitud serializada de JSON para cualquiera de los tipos de datos admitidos, como NumPy, Pandas y PySpark. Es posible agrupar por lotes a varios eventos de entrada en una única solicitud.

La siguiente consulta de Stream Analytics es un ejemplo de cómo invocar una función definida por el usuario de Azure Machine Learning:

```SQL
SELECT udf.score(<model-specific-data-structure>)
INTO output
FROM input
```

Stream Analytics solo admite la inclusión de un parámetro para las funciones de Azure Machine Learning. Quizá deba preparar los datos antes de pasarlos como entrada a una función definida por el usuario de Machine Learning.

## <a name="pass-multiple-input-parameters-to-the-udf"></a>Pasar varios parámetros de entrada a la función definida por el usuario

Los ejemplos más comunes de entradas para los modelos de aprendizaje automático son los DataFrames y las matrices de NumPy. Puede crear una matriz mediante una función definida por el usuario de JavaScript; también puede crear un DataFrame serializado con JSON mediante la cláusula `WITH`.

### <a name="create-an-input-array"></a>Creación de una matriz de entradas

Puede crear una función definida por el usuario de JavaScript que acepte un número *N* de entradas y cree una matriz que se puede usar como entrada para la función definida por el usuario de Azure Machine Learning.

```javascript
function createArray(vendorid, weekday, pickuphour, passenger, distance) {
    'use strict';
    var array = [vendorid, weekday, pickuphour, passenger, distance]
    return array;
}
```

Una vez que haya agregado la función definida por el usuario de JavaScript a su trabajo, puede invocar la función definida por el usuario de Azure Machine Learning mediante la siguiente consulta:

```SQL
SELECT udf.score(
udf.createArray(vendorid, weekday, pickuphour, passenger, distance)
)
INTO output
FROM input
```

El siguiente código JSON es una solicitud de ejemplo:

```JSON
{
    "data": [
        ["1","Mon","12","1","5.8"],
        ["2","Wed","10","2","10"]
    ]
}
```

### <a name="create-a-pandas-or-pyspark-dataframe"></a>Creación de un DataFrame de Pandas o PySpark

Puede usar la cláusula `WITH` para crear un DataFrame serializado de JSON que se pueda pasar como entrada para la función definida por el usuario de Azure Machine Learning, tal como se muestra a continuación.

La siguiente consulta crea un DataFrame al seleccionar los campos necesarios y usa dicho DataFrame como entrada para la función definida por el usuario de Azure Machine Learning.

```SQL
WITH 
Dataframe AS (
SELECT vendorid, weekday, pickuphour, passenger, distance
FROM input
)

SELECT udf.score(Dataframe)
INTO output
FROM input
```

El siguiente código JSON es una solicitud de ejemplo de la consulta anterior:

```JSON
{
    "data": [{
            "vendorid": "1",
            "weekday": "Mon",
            "pickuphour": "12",
            "passenger": "1",
            "distance": "5.8"
        }, {
            "vendorid": "2",
            "weekday": "Tue",
            "pickuphour": "10",
            "passenger": "2",
            "distance": "10"
        }
    ]
}
```

## <a name="optimize-the-performance-for-azure-machine-learning-udfs"></a>Optimización del rendimiento de las funciones definidas por el usuario de Azure Machine Learning

Al implementar el modelo en Azure Kubernetes Service, puede [generar un perfil de modelo para determinar el uso de los recursos](../machine-learning/how-to-deploy-profile-model.md). También puede [habilitar App Insights para sus implementaciones](../machine-learning/how-to-enable-app-insights.md). De este modo, aprenderá sobre las tasas de solicitudes, los tiempos de respuesta y las tasas de error.

Si tiene un escenario con un alto procesamiento de eventos, quizá necesite cambiar los siguientes parámetros en Stream Analytics para lograr un rendimiento óptimo con latencias bajas de un extremo a otro:

1. Número máximo de lotes.
2. Número de solicitudes paralelas por partición.

### <a name="determine-the-right-batch-size"></a>Determinación del tamaño de lote adecuado

Una vez que haya implementado el servicio web, envíe una solicitud de ejemplo con distintos tamaños de lote a partir de 50 y aumente la cifra en grupos de cien. Por ejemplo, 200, 500, 1000, 2000, etc. Verá que, después de un tamaño de lote determinado, aumentará la latencia de la respuesta. El punto en el que la latencia de la respuesta aumente debe indicar el número máximo de lotes para el trabajo.

### <a name="determine-the-number-of-parallel-requests-per-partition"></a>Definición del número de solicitudes paralelas por partición

En condiciones de escalado óptimas, el trabajo de Stream Analytics debe ser capaz de enviar varias solicitudes paralelas al servicio web y obtener una respuesta en unos pocos milisegundos. La latencia de la respuesta del servicio web puede afectar de forma directa a la latencia y el rendimiento del trabajo de Stream Analytics. Si la llamada del trabajo al servicio web tarda mucho tiempo, probablemente notará un aumento en el retraso de la marca de agua, además de un aumento en el número de eventos de entrada pendientes.

Para evitar este tipo de latencia, asegúrese de que el clúster de Azure Kubernetes Service (AKS) se aprovisionó con el [número correcto de nodos y réplicas](../machine-learning/how-to-deploy-azure-kubernetes-service.md#using-the-cli). Es fundamental que el servicio web tenga alta disponibilidad y devuelva respuestas correctas. Si el trabajo recibe una respuesta de Servicio no disponible (503) de parte del servicio web, la operación se reintentará de forma continua con retroceso exponencial. Cualquier respuesta que no sea Correcto (200) y Servicio no disponible (503) hará que el trabajo cambie a un estado de error.

## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Funciones definidas por el usuario en JavaScript para Azure Stream Analytics](stream-analytics-javascript-user-defined-functions.md)
* [Escalado del trabajo de Stream Analytics con funciones de Azure Machine Learning Studio (clásico)](stream-analytics-scale-with-machine-learning-functions.md)

