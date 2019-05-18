---
title: Explorar y preparar los datos (clase de conjunto de datos)
titleSuffix: Azure Machine Learning service
description: Explorar datos con las estadísticas de resumen y preparar los datos a través de ingeniería de características, transformación y limpieza de datos
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sihhu
author: MayMSFT
manager: cgronlun
ms.reviewer: nibaccam
ms.date: 05/02/19
ms.openlocfilehash: 70712605cc97670b625d32052bb79b4a666e4281
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65603160"
---
# <a name="explore-and-prepare-data-with-the-dataset-class-preview"></a>Explorar y preparar los datos con la clase de conjunto de datos (versión preliminar)

Obtenga información sobre cómo explorar y preparar los datos con el [SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py). El [Dataset](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py) clase (versión preliminar) le permite explorar y preparar los datos al proporcionar funciones tales como: muestreo, las estadísticas de resumen y transformaciones inteligentes. Los pasos de transformación se guardan en [definiciones de conjunto de datos](how-to-manage-dataset-definitions.md) con la capacidad de controlar varios archivos grandes de distintos esquemas de una manera altamente escalable.

> [!Important]
> Algunas clases de conjunto de datos (versión preliminar) tienen dependencias en el [azureml dataprep](https://docs.microsoft.com/python/api/azureml-dataprep/?view=azure-ml-py) (GA) del paquete. Mientras que las funciones de transformación pueden realizarse directamente con el GA'ed [funciones de preparación de datos](how-to-transform-data.md), se recomienda que los contenedores del paquete de conjunto de datos descritos en este artículo, si está creando una nueva solución. Azure Machine Learning conjuntos de datos (versión preliminar) le permiten transformar no solo los datos, sino también [datos de instantánea](how-to-create-dataset-snapshots.md) y almacenar [definiciones de conjunto de datos con control de versiones](how-to-manage-dataset-definitions.md). Los conjuntos de datos es la siguiente versión del SDK de preparación de datos, que ofrece funcionalidad ampliada para administrar conjuntos de datos en soluciones de inteligencia artificial.

## <a name="prerequisites"></a>Requisitos previos

Para explorar y preparar los datos, necesitará:

* Una suscripción de Azure. Si no tiene una suscripción a Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning Service](https://aka.ms/AMLFree).

* Un área de trabajo de Azure Machine Learning. Consulte [crear un área de trabajo del servicio de Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/service/setup-create-workspace).

* El SDK de Azure Machine Learning para Python (versión 1.0.21 o posterior). Para instalar o actualizar a la versión más reciente del SDK, consulte [instalar o actualizar el SDK](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py).

* La preparación de datos de aprendizaje de máquina de Azure SDK. Para instalar o actualizar a la versión más reciente, consulte [instalar o actualizar el SDK de preparación de datos](https://docs.microsoft.com/python/api/overview/azure/dataprep/intro?view=azure-dataprep-py#install).

* Descargar los archivos de ejemplo para seguir los ejemplos: [crime.csv](https://dprepdata.blob.core.windows.net/dataset-sample-files/crime.csv) y [city.json](https://dprepdata.blob.core.windows.net/dataset-sample-files/city.json).

## <a name="sampling"></a>muestreo

Toma una muestra de los datos para obtener una comprensión inicial de la arquitectura de datos y su contenido. En este momento, el [ `sample()` ](https://docs.microsoft.com//python/api/azureml-core/azureml.core.dataset(class)?view=azure-ml-py#sample-sample-strategy--arguments-) estrategias de muestreo Top N aleatorio Simple y Stratified admite el método de la clase de conjunto de datos.

```Python
from azureml.core.dataset import Dataset
import random

# create an in-memory Dataset from a local file
dataset = Dataset.auto_read_files('./data/crime.csv')

# create seed for Simple Random and Stratified sampling
seed = random.randint(0, 4294967295)
```

### <a name="top-n-sample"></a>Ejemplo de las primeras N

Para el muestreo de Top N, los n primeros registros del conjunto de datos son su ejemplo. Esto resulta útil si simplemente desea hacerse una idea de lo que el aspecto de los registros de datos, como o para ver cuáles son los campos en los datos.

```Python
top_n_sample_dataset = dataset.sample('top_n', {'n': 5})
top_n_sample_dataset.to_pandas_dataframe()
```

||ID|Case Number|Date|Bloquear|IUCR|Primary Type|...|
-|--|-----------|----|-----|----|------------|---
0|10498554|HZ239907|4/4/2016 23:56|007XX E 111TH ST|1153|PRÁCTICA ENGAÑOSA|...
1|10516598|HZ258664|4/15/2016 17:00|082XX S MARSHFIELD AVE|890|THEFT|...
2|10519196|HZ261252|4/15/2016 10:00|104XX S SACRAMENTO AVE|1154|PRÁCTICA ENGAÑOSA|...
3|10519591|HZ261534|4/15/2016 9:00|113XX UARDAR ASUSTADOS|1120|PRÁCTICA ENGAÑOSA|...
4|10534446|HZ277630|4/15/2016 10:00|055XX N KEDZIE AVE|890|THEFT|...

### <a name="simple-random-sample"></a>Muestra aleatoria simple

En el muestreo aleatorio Simple, todos los miembros de la población de datos tienen la misma probabilidad de ser seleccionados como parte del ejemplo. En el `simple_random` ejemplo de estrategia, los registros del conjunto de datos se seleccionan en función de la probabilidad especificada y devuelve un conjunto de datos modificado. El parámetro de inicialización es opcional.

```Python
simple_random_sample_dataset = dataset.sample('simple_random', {'probability':0.3, 'seed': seed})
simple_random_sample_dataset.to_pandas_dataframe()
```

||ID|Case Number|Date|Bloquear|IUCR|Primary Type|...|
-|--|-----------|----|-----|----|------------|---
0|10516598|HZ258664|4/15/2016 17:00|082XX S MARSHFIELD AVE|890|THEFT|...
1|10519196|HZ261252|4/15/2016 10:00|104XX S SACRAMENTO AVE|1154|PRÁCTICA ENGAÑOSA|...
2|10534446|HZ277630|4/15/2016 10:00|055XX N KEDZIE AVE|890|THEFT|...

### <a name="stratified-sample"></a>Ejemplo estratificado

Ejemplos de estratificado Asegúrese de que ciertos grupos de una población se representan en el ejemplo. En el `stratified` ejemplo de estrategia, la población se divide en estratos o subgrupos, basándose en similitudes, y los registros se seleccionan aleatoriamente de cada estrato según los pesos de estratos indicados por el `fractions` parámetro.

En el ejemplo siguiente, que cada registro de grupo por las columnas especificadas y que incluye dicho registro basándose en la información de peso estratos X `fractions`. Si no se especifica un estratos o el registro no se pueden agrupar, el peso predeterminado para el ejemplo es 0.

```Python
# take 50% of records with `Primary Type` as `THEFT` and 20% of records with `Primary Type` as `DECEPTIVE PRACTICE` into sample Dataset
fractions = {}
fractions[('THEFT',)] = 0.5
fractions[('DECEPTIVE PRACTICE',)] = 0.2

sample_dataset = dataset.sample('stratified', {'columns': ['Primary Type'], 'fractions': fractions, 'seed': seed})

sample_dataset.to_pandas_dataframe()
```

||ID|Case Number|Date|Bloquear|IUCR|Primary Type|...|
-|--|-----------|----|-----|----|------------|---
0|10516598|HZ258664|4/15/2016 17:00|082XX S MARSHFIELD AVE|890|THEFT|...
1|10534446|HZ277630|4/15/2016 10:00|055XX N KEDZIE AVE|890|THEFT|...
2|10535059|HZ278872|4/15/2016 4:30|004XX UARDAR KILBOURN|810|THEFT|...

## <a name="explore-with-summary-statistics"></a>Explorar con estadísticas de resumen

 Detectar anomalías, faltan valores, o los recuentos de errores con el [ `get_profile()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#get-profile-arguments-none--generate-if-not-exist-true--workspace-none--compute-target-none-) método. Esta función obtiene el perfil y estadísticas de resumen de los datos, que a su vez le ayuda a determinan las operaciones de preparación de datos necesarios para aplicar.

```Python
dataset.get_profile()
```

||Type|Min|max|Recuento|Falta el recuento|No falta el recuento|Falta el porcentaje|Recuento de errores|Recuento vacío|Cuantil 0,1 %|Cuantil 1 %|Cuantil 5 %|Cuantil 25 %|Cuantil 50 %|Cuantil 75 %|Cuantil 95 %|Cuantil 99 %|Cuantil 99,9 %|Media|Desviación estándar|Variance|Asimetría|Curtosis
-|----|---|---|-----|-------------|-----------------|---------------|-----------|-----------|-------------|-----------|-----------|------------|------------|------------|------------|------------|--------------|----|------------------|--------|--------|--------
ID|FieldType.INTEGER|1.04986e+07|1.05351e+07|10.0|0.0|10.0|0.0|0.0|0.0|1.04986e+07|1.04992e+07|1.04986e+07|1.05166e+07|1.05209e+07|1.05259e+07|1.05351e+07|1.05351e+07|1.05351e+07|1.05195e+07|12302.7|1.51358e+08|-0.495701|-1.02814
Case Number|FieldType.STRING|HZ239907|HZ278872|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
Date|FieldType.DATE|2016-04-04 23:56:00+00:00|2016-04-15 17:00:00+00:00|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
Bloquear|FieldType.STRING|004XX UARDAR KILBOURN|113XX UARDAR ASUSTADOS|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
IUCR|FieldType.INTEGER|810|1154|10.0|0.0|10.0|0.0|0.0|0.0|810|850|810|890|1136|1153|1154|1154|1154|1058.5|137.285|18847.2|-0.785501|-1.3543
Primary Type|FieldType.STRING|PRÁCTICA ENGAÑOSA|THEFT|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
DESCRIPCIÓN|FieldType.STRING|COMPROBACIÓN DE FANTASMA|A TRAVÉS DE 500 USD|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
Location Description|FieldType.STRING||SCHOOL, PÚBLICO, CREAR|10.0|0.0|10.0|0.0|0.0|1.0||||||||||||||
Arrest|FieldType.BOOLEAN|False|False|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
Domestic|FieldType.BOOLEAN|False|False|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
latido|FieldType.INTEGER|531|2433|10.0|0.0|10.0|0.0|0.0|0.0|531|531|531|614|1318.5|1911|2433|2433|2433|1371.1|692.094|478994|0.105418|-1.60684
Distrito|FieldType.INTEGER|5|24|10.0|0.0|10.0|0.0|0.0|0.0|5|5|5|6|13|19|24|24|24|13.5|6.94822|48.2778|0.0930109|-1.62325
Ward|FieldType.INTEGER|1|48|10.0|0.0|10.0|0.0|0.0|0.0|1|5|1|9|22.5|40|48|48|48|24,5|16.2635|264.5|0.173723|-1.51271
Community Area|FieldType.INTEGER|4|77|10.0|0.0|10.0|0.0|0.0|0.0|4|8.5|4|24|37.5|71|77|77|77|41.2|26.6366|709.511|0.112157|-1.73379
FBI Code|FieldType.INTEGER|6|11|10.0|0.0|10.0|0.0|0.0|0.0|6|6|6|6|11|11|11|11|11|9.4|2.36643|5.6|-0.702685|-1.59582
X Coordinate|FieldType.INTEGER|1.16309e+06|1.18336e+06|10.0|7.0|3.0|0.7|0.0|0.0|1.16309e+06|1.16309e+06|1.16309e+06|1.16401e+06|1.16678e+06|1.17921e+06|1.18336e+06|1.18336e+06|1.18336e+06|1.17108e+06|10793.5|1.165e+08|0.335126|-2.33333
Y Coordinate|FieldType.INTEGER|1.8315e+06|1.908e+06|10.0|7.0|3.0|0.7|0.0|0.0|1.8315e+06|1.8315e+06|1.8315e+06|1.83614e+06|1.85005e+06|1.89352e+06|1.908e+06|1.908e+06|1.908e+06|1.86319e+06|39905.2|1.59243e+09|0.293465|-2.33333
Year|FieldType.INTEGER|2016|2016|10.0|0.0|10.0|0.0|0.0|0.0|2016|2016|2016|2016|2016|2016|2016|2016|2016|2016|0|0|NaN|NaN
Updated On|FieldType.DATE|2016-05-11 15:48:00+00:00|2016-05-27 15:45:00+00:00|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
Latitud|FieldType.DECIMAL|41.6928|41.9032|10.0|7.0|3.0|0.7|0.0|0.0|41.6928|41.6928|41.6928|41.7057|41.7441|41.8634|41.9032|41.9032|41.9032|41.78|0.109695|0.012033|0.292478|-2.33333
Longitud|FieldType.DECIMAL|-87.6764|-87.6043|10.0|7.0|3.0|0.7|0.0|0.0|-87.6764|-87.6764|-87.6764|-87.6734|-87.6645|-87.6194|-87.6043|-87.6043|-87.6043|-87.6484|0.0386264|0.001492|0.344429|-2.33333
Ubicación|FieldType.STRING||(41.903206037, -87.676361925)|10.0|0.0|10.0|0.0|0.0|7.0||||||||||||||

## <a name="impute-missing-values"></a>Atribución de valores que faltan

En conjuntos de datos, los valores null, NaN y los valores que no incluyen contenido se consideran valores que faltan. Estos pueden afectar al rendimiento de los modelos de aprendizaje automático o conducir a conclusiones no válidas. Imputación es un enfoque común para tratar los valores que faltan y es útil cuando tiene un porcentaje alto de falta el valor de los registros que simplemente no se puede eliminar.

Desde el perfil del conjunto de datos generado en la sección anterior, vemos que `Latitude` y `Longitude` columnas tienen un alto porcentaje de valores que faltan. En este ejemplo, se calculará la media e imputar valores ausentes para esas dos columnas.

En primer lugar, obtenga la última definición del conjunto de datos con [ `get_definition()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#get-definition-version-id-none-) y reducir los datos con [ `keep_columns()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#keep-columns-columns--multicolumnselection-----azureml-dataprep-api-dataflow-dataflow), por lo que nos ver solo las columnas que queremos dirección.

```Python
from azureml.core.dataset import Dataset
import azureml.dataprep as dprep

# get the latest definition of Dataset
ds_def = dataset.get_definition()

# keep useful columns for this example
ds_def = ds_def.keep_columns(['ID', 'Arrest', 'Latitude', 'Longitude'])
ds_def.head(3)
```

||ID|Arrest| Latitud|Longitud|
-|---------|-----|---------|----------|
|0|10498554|False|41.692834|-87.604319|
|1|10516598|False| 41.744107 |-87.664494|
|2|10519196|False| NaN|NaN|

A continuación, compruebe el `MEAN` valor de la columna de latitud utilizando el [ `summarize()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#summarize-summary-columns--typing-union-typing-list-azureml-dataprep-api-dataflow-summarycolumnsvalue---nonetype----none--group-by-columns--typing-union-typing-list-str---nonetype----none--join-back--bool---false--join-back-columns-prefix--typing-union-str--nonetype----none-----azureml-dataprep-api-dataflow-dataflow) función. Esta función acepta una matriz de columnas en el parámetro `group_by_columns` para especificar el nivel de agregación. El `summary_columns` parámetro acepta la `SummaryColumnsValue` función, que especifica el nombre de columna actual, el nuevo nombre del campo calculado y el `SummaryFunction` para realizar.

```Python
lat_mean = ds_def.summarize(group_by_columns = ['Arrest'],
                            summary_columns = [dprep.SummaryColumnsValue(column_id = 'Latitude',
                                                                         summary_column_name = 'Latitude_MEAN',
                                                                         summary_function = dprep.SummaryFunction.MEAN)])
lat_mean = lat_mean.filter(lat_mean['Arrest'] == False)
lat_mean.head(1)
```

||Arrest|Latitude_MEAN|
--|-----|--------|
|0|False|41.780049|

Una vez que comprobemos los valores para atribuir, utilice [ `ImputeMissingValuesBuilder` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.api.builders.imputemissingvaluesbuilder?view=azure-dataprep-py) para obtener información sobre una expresión fija que imputes las columnas con un calculado `MIN`, `MAX`, `MEAN` valor, o un `CUSTOM` valor. Cuando se especifica `group_by_columns`, se atribuirán los valores que faltan por grupo a `MIN`, `MAX` y `MEAN` calculados por grupo.

El [ `ImputeColumnArguments` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.api.builders.imputecolumnarguments?view=azure-dataprep-pyfunction) acepta una cadena column_id y un `ReplaceValueFunction` para especificar el tipo de impute. Para el valor de longitud que falta, imputar con-87 en función de información externos.

```Python
# impute with MEAN
impute_mean = dprep.ImputeColumnArguments(column_id = 'Latitude',
                                          impute_function = dprep.ReplaceValueFunction.MEAN)

# impute with custom value -87
impute_custom = dprep.ImputeColumnArguments(column_id='Longitude',
                                            custom_impute_value=-87)
```

Imputar pasos se pueden encadenar juntas en un `ImputeMissingValuesBuilder` objeto utilizando el [ `Builders` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.api.builders.builders?view=azure-dataprep-py) clase función [ `impute_missing_values()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.api.builders.builders?view=azure-dataprep-py#impute-missing-values-impute-columns--typing-list-azureml-dataprep-api-builders-imputecolumnarguments----none--group-by-columns--typing-union-typing-list-str---nonetype----none-----azureml-dataprep-api-builders-imputemissingvaluesbuilder). El parámetro `impute_columns` acepta una matriz de objetos `ImputeColumnArguments`.

```Python
# get instance of ImputeMissingValuesBuilder
impute_builder = ds_def.builders.impute_missing_values(impute_columns=[impute_mean, impute_custom],
                                                   group_by_columns=['Arrest'])
```

Llame a la [ `learn()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.api.builders.imputemissingvaluesbuilder?view=azure-dataprep-py#learn------none) función para almacenar los pasos impute y aplicarlos a un objeto de flujo de datos mediante [ `to_dataflow()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.api.builders.imputemissingvaluesbuilder?view=azure-dataprep-py#to-dataflow------azureml-dataprep-api-dataflow-dataflow).

```Python
impute_builder.learn()
ds_def = impute_builder.to_dataflow()
ds_def.head(3)
```

Como se muestra en la siguiente tabla de salida, la latitud que faltan se imputen con el `MEAN` valor `Arrest==False` grupo y la longitud que faltan se imputen con-87.

||ID|Arrest|Latitud|Longitud
-|---------|-----|---------|----------
0|10498554|False|41.692834|-87.604319
1|10516598|False|41.744107|-87.664494
2|10519196|False|41.780049|-87.000000

Actualizar la definición del conjunto de datos, [ `update_definition()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset(class)?view=azure-ml-py#update-definition-definition--definition-update-message-) para mantener los pasos de transformación realizada.

```Python
dataset = dataset.update_definition(ds_def, 'Impute Missing')
dataset.head(3)
```

||ID|Arrest|Latitud|Longitud
-|---------|-----|---------|----------
0|10498554|False|41.692834|-87.604319
1|10516598|False|41.744107|-87.664494
2|10519196|False|41.780049|-87.000000

## <a name="create-assertion-rules"></a>Crear reglas de aserción

Con frecuencia, los datos a la que trabajamos con durante la limpieza y preparación de datos es solo un subconjunto del total de datos que necesitamos para producción. Como resultado, algunas de las suposiciones que tomamos como parte de nuestro proceso de limpieza es posible que resultó para ser false. Por ejemplo, en un conjunto de datos que se actualizan continuamente, una columna que contenía originalmente sólo números dentro de un intervalo determinado podría contener una gama más amplia de valores de las ejecuciones posteriores. Estos errores suelen dar lugar a canalizaciones rotas o datos incorrectos.

Creando las aserciones en datos, que se evalúan a medida que se ejecuta la canalización de compatibilidad con conjuntos de datos. Estas aserciones nos permiten comprobar que nuestra suposiciones sobre los datos siguen sea precisa y, si no, para controlar los errores en consecuencia.

Por ejemplo, si desea restringir `Latitude` y `Longitude` valores del conjunto de datos a intervalos numéricos específicos, el [ `assert_value()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#assert-value-columns--multicolumnselection--expression--azureml-dataprep-api-expressions-expression--policy--azureml-dataprep-api-engineapi-typedefinitions-assertpolicy----assertpolicy-errorvalue--1---error-code--str----assertionfailed------azureml-dataprep-api-dataflow-dataflow) método garantiza que siempre es así.

```Python
from azureml.dataprep import value
from azureml.core.dataset import Dataset

# get the latest definition of the Dataset
ds_def = dataset.get_definition()

# set assertion rules for `Latitude` and `Longitude` columns
ds_def = ds_def.assert_value('Latitude', (value <= 90) & (value >= -90), error_code='InvalidLatitude')
ds_def = ds_def.assert_value('Longitude', (value <= 180) & (value >= -87), error_code='InvalidLongitude')

ds_def.get_profile()
```

||Type|Min|max|Recuento|Falta el recuento|No falta el recuento|Falta el porcentaje|Recuento de errores|Recuento vacío|Cuantil 0,1 %|Cuantil 1 %|Cuantil 5 %|Cuantil 25 %|Cuantil 50 %|Cuantil 75 %|Cuantil 95 %|Cuantil 99 %|Cuantil 99,9 %|Media|Desviación estándar|Variance|Asimetría|Curtosis
-|----|---|---|-----|-------------|-----------------|---------------|-----------|-----------|-------------|-----------|-----------|------------|------------|------------|------------|------------|--------------|----|------------------|--------|--------|--------
ID|FieldType.INTEGER|1.04986e+07|1.05351e+07|10.0|0.0|10.0|0.0|0.0|0.0|1.04986e+07|1.04992e+07|1.04986e+07|1.05166e+07|1.05209e+07|1.05259e+07|1.05351e+07|1.05351e+07|1.05351e+07|1.05195e+07|12302.7|1.51358e+08|-0.495701|-1.02814
Arrest|FieldType.BOOLEAN|False|False|10.0|0.0|10.0|0.0|0.0|0.0||||||||||||||
Latitud|FieldType.DECIMAL|41.6928|41.9032|10.0|0.0|10.0|0.0|0.0|0.0|41.6928|41.7185|41.6928|41.78|41.78|41.78|41.9032|41.9032|41.9032|41.78|0.0517107|0.002674|0.837593|1.05
Longitud|FieldType.INTEGER|-87|-87|10.0|0.0|10.0|0.0|3.0|0.0|-87|-87|-87|-87|-87|-87|-87|-87|-87|-87|0|0|NaN|NaN

Desde el perfil, verá que el `Error Count` para el `Longitude` columna es 3. El siguiente código filtra el conjunto de datos, recupera el error y ve qué valor hace que la aserción de un error. Desde aquí, ajuste el código y limpiar los datos de forma adecuada.

```Python
from azureml.dataprep import col

ds_error = ds_def.filter(col('Longitude').is_error())
error = ds_error.head(10)['Longitude'][0]

print(error.originalValue)
```

    -87.60431945

## <a name="derive-columns-by-example"></a>Derivar columnas por ejemplos

Una de las herramientas más avanzadas para conjuntos de datos es la capacidad para derivar columnas mediante ejemplos de resultados deseados. Esto le permite dar el SDK de un ejemplo, para que pueda generar código para conseguir las transformaciones previstas.

```Python
from azureml.core.dataset import Dataset

# create an in-memory Dataset from a local file
dataset = Dataset.auto_read_files('./data/crime.csv')
dataset.head(3)
```

||ID|Case Number|Date|Bloquear|...|
-|---------|-----|---------|----|---
0|10498554|HZ239907|2016-04-04 23:56:00|007XX E 111TH ST|...
1|10516598|HZ258664|2016-04-15 17:00:00|082XX S MARSHFIELD AVE|...
2|10519196|HZ261252|2016-04-15 10:00:00|104XX S SACRAMENTO AVE|...

Digamos que necesita transformar el formato de fecha y hora a ' 2016-04-04 PM de 10-12 AM'. En el [ `derive_column_by_example()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#derive-column-by-example-source-columns--sourcecolumns--new-column-name--str--example-data--exampledata-----azureml-dataprep-api-dataflow-dataflow) argumento, se proporcionan ejemplos de su resultado deseado en el `example_data` parámetro en este formato: *(salida original, el resultado deseado)*.

El código siguiente proporciona dos ejemplos de salida deseado, ("2016-04-04 23:56:00", "2016-04-04 10 PM-12 a. M.") y ("2016-04-15 17:00:00", "2016-04-15 4 p. M. - 6 p. M.")

```Python
ds_def = dataset.get_definition()
ds_def = ds_def.derive_column_by_example(
        source_columns = "Date",
        new_column_name = "Date_Time_Range",
        example_data = [("2016-04-04 23:56:00", "2016-04-04 10PM-12AM"), ("2016-04-15 17:00:00", "2016-04-15 4PM-6PM")]
    )
ds_def.keep_columns(['ID','Date','Date_Time_Range']).head(3)
```

En la tabla siguiente, tenga en cuenta que una nueva columna, Date_Time_Range contiene registros en el formato especificado.

||ID|Date|Date_Time_Range
-|--------|-----|----
0|10498554|2016-04-04 23:56:00|2016-04-04 10 PM-12 A. M.
1|10516598|2016-04-15 17:00:00|2016-04-15 P.M. DE 4 A 6 P.M.
2|10519196|2016-04-15 10:00:00|2016-04-15 10 A.M. A 12 P.M.

```Python
# update Dataset definition to keep the transformation steps performed.
dataset = dataset.update_definition(ds_def, 'Derive Date_Time_Range')
```

## <a name="fuzzy-groupings"></a>Agrupaciones aproximadas

Al recopilar datos de orígenes diferentes pueden producirse las variaciones de ortografía, mayúsculas y minúsculas o abreviaturas de las mismas entidades. Estandarizar y conciliar estas variantes con la funcionalidad de agrupación aproximada y agrupación en clústeres de texto, del SDK automáticamente.

Por ejemplo, la columna `inspections.business.city` contiene varias formas de que el nombre de ciudad "San Francisco".

```Python
from azureml.core.dataset import Dataset

# create an in-memory Dataset from a local json file
dataset = Dataset.auto_read_files('./data/city.json')
dataset.head(5)
```

||inspections.business.business_id|inspections.business_name|inspections.business.address|inspections.business.city|...|
-|-----|-------------------------|------------|--|---
0|16162|Quick-N-Ezee indios alimentos|3861 24th St|SF|...
1|67565|Rey de café fideos tailandés|1541 TARAVAL St|SAN FRANCISCO|...
2|67565|Rey de café fideos tailandés|1541 TARAVAL St|SAN FRANCISCO|...
3|68701|Grindz|832 Clement St|SF|...
4|69186|Eventos, Inc. & encargándose Premier|1255 22nd St|S.F.|...

Vamos a usar el [ `fuzzy_group_column()` ](https://docs.microsoft.com/python/api/azureml-dataprep/azureml.dataprep.dataflow?view=azure-dataprep-py#fuzzy-group-column-source-column--str--new-column-name--str--similarity-threshold--float---0-8--similarity-score-column-name--str---none-----azureml-dataprep-api-dataflow-dataflow) método para agregar una columna con la forma canónica detectada automáticamente de "San Francisco". Los argumentos `source_column` y `new_column_name` son necesarios. También tiene la opción:

* Crea una nueva columna, `similarity_score_column_name`, que muestra la puntuación de similitud entre cada par de valores originales y canónicos.

* Proporcione un `similarity_threshold`, que es la puntuación de similitud mínima para los valores que se agrupan juntos.

```Python
# get the latest Dataset definition
ds_def = dataset.get_definition()
ds_def = ds_def.fuzzy_group_column(source_column='inspections.business.city',
                                       new_column_name='city_grouped',
                                       similarity_threshold=0.8,
                                       similarity_score_column_name='similarity_score')
ds_def.head(5)
```

||inspections.business.business_id|inspections.business_name|inspections.business.address|inspections.business.city|city_grouped|similarity_score|...|
-|-----|-------------------------|------------|--|---|---|---
0|16162|Quick-N-Ezee indios alimentos|3861 24th St|SF|San Francisco|0.814806|...
1|67565|Rey de café fideos tailandés|1541 TARAVAL St|SAN FRANCISCO|San Francisco|1.000000|...
2|67565|Rey de café fideos tailandés|1541 TARAVAL St|SAN FRANCISCO|San Francisco|1.000000|...
3|68701|Grindz|832 Clement St|SF|San Francisco|0.814806|...
4|69186|Eventos, Inc. & encargándose Premier|1255 22nd St|S.F.|San Francisco|0.814806|...

En la definición de conjunto de datos resultante, todas las distintas variaciones de representar "San Francisco" en los datos se han normalizado a la misma cadena, "San Francisco".

Guardar este paso de agrupación aproximada en la definición del conjunto de datos más reciente con `update_definition()`.

```Python
dataset = dataset.update_definition(ds_def, 'fuzzy grouping')
```

## <a name="next-steps"></a>Pasos siguientes

* [Administrar el ciclo de vida de las definiciones de conjunto de datos](how-to-manage-dataset-definitions.md).

* Consulte el automatizada machine learning [tutorial](tutorial-auto-train-models.md) para obtener un ejemplo de modelo de regresión.

* Consulte el SDK de [Introducción](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py) para obtener ejemplos de uso y patrones de diseño.

* Para obtener un ejemplo del uso de conjuntos de datos, vea el [cuadernos de ejemplo](https://aka.ms/dataset-tutorial).