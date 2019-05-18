---
title: Cree y registre los conjuntos de datos con el área de trabajo
titleSuffix: Azure Machine Learning service
description: Obtenga información sobre cómo crear conjuntos de datos desde diversos orígenes y registre los conjuntos de datos con el área de trabajo
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sihhu
author: MayMSFT
manager: cgronlun
ms.reviewer: nibaccam
ms.date: 05/02/19
ms.openlocfilehash: d3502219f03d4ad076a693ab990f2fadb0b5d558
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65800829"
---
# <a name="create-and-register-azure-machine-learning-datasets-preview"></a>Crear y registrar los conjuntos de datos de Azure Machine Learning (versión preliminar)

En este artículo, obtendrá información sobre los flujos de trabajo de Azure Machine Learning para crear y registrar los conjuntos de datos y cómo tener acceso a ellos para su reutilización entre experimentos locales y remotos.

Azure Machine Learning conjuntos de datos (versión preliminar) facilitan el acceso y trabajar con los datos. Los conjuntos de datos administración datos en diversos escenarios, como el entrenamiento del modelo y crear la canalización. Mediante el [SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py), puede trabajar con datos en formatos populares, tener acceso al almacenamiento subyacente, explorar y preparar datos, administrar el ciclo de vida de las definiciones de conjunto de datos diferente y comparar entre conjuntos de datos usados en entrenamiento y de producción.

## <a name="prerequisites"></a>Requisitos previos

Para crear y registrar los conjuntos de datos, es preciso:

* Una suscripción de Azure. Si no tiene una suscripción a Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning Service](https://aka.ms/AMLFree).

* Un área de trabajo de Azure Machine Learning. Consulte [crear un área de trabajo del servicio de Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/service/setup-create-workspace).

* Azure Machine Learning SDK para Python. Para instalar o actualizar a la versión más reciente del SDK, consulte [instalar o actualizar el SDK](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py).

> [!Note]
> Algunas clases de conjunto de datos (versión preliminar) tienen dependencias en el [azureml dataprep](https://docs.microsoft.com/python/api/azureml-dataprep/?view=azure-ml-py) (GA) del paquete. Los usuarios de Linux, estas clases solo se admiten en las siguientes distribuciones:  Red Hat Enterprise Linux, Ubuntu, Fedora y CentOS.

## <a name="create-datasets-from-local-files"></a>Crear conjuntos de datos desde archivos locales

Cargar archivos desde el equipo local mediante la especificación de la ruta de acceso de archivo o carpeta con el [ `auto_read_files()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset(class)?view=azure-ml-py#auto-read-files-path--include-path-false-) método desde el `Dataset` clase.  Este método lleva a cabo los siguientes pasos sin necesidad de especificar el tipo de archivo o analizar los argumentos:

* Inferir y estableciendo el delimitador.
* Omitiendo los registros vacíos en la parte superior del archivo.
* Inferir y la configuración de la fila de encabezado.
* Inferir y conversión de tipos de datos de columna.

```Python
from azureml.core.dataset import Dataset

dataset = Dataset.auto_read_files('./data/crime.csv')
```

Como alternativa, puede usar las funciones específicas del archivo para controlar explícitamente el análisis del archivo. Actualmente, el SDK de Datasets admite [delimitados](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#from-delimited-files-path--separator------header--promoteheadersbehavior-allfileshavesameheaders--3---encoding--fileencoding-utf8--0---quoting-false--infer-column-types-true--skip-rows-0--skip-mode--skiplinesbehavior-norows--0---comment-none--include-path-false--archive-options-none-), [Excel](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#from-excel-files-path--sheet-name-none--use-column-headers-false--skip-rows-0--include-path-false--infer-column-types-true-), [Parquet](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#from-parquet-files-path--include-path-false-), [binario](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#from-binary-files-path-), y [json](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#from-json-files-path--encoding--fileencoding-utf8--0---flatten-nested-arrays-false--include-path-false-) formatos de archivo.

## <a name="create-datasets-from-azure-datastores"></a>Crear conjuntos de datos desde almacenes de datos de Azure

Para crear conjuntos de datos desde un almacén de datos de Azure, no olvide:

* Compruebe que dispone de colaborador o propietario de tener acceso al almacén de datos de Azure registrados.

* Importar el [ `Workspace` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py) y [ `Datastore` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.datastore(class)?view=azure-ml-py#definition) y `Dataset` paquetes desde el SDK.

```Python
from azureml.core.workspace import Workspace
from azureml.core.datastore import Datastore
from azureml.core.dataset import Dataset

datastore_name = 'your datastore name'

# get existing workspace
workspace = Workspace.from_config()
```

 El `get()` método recupera un almacén de datos existente en el área de trabajo.

```
dstore = Datastore.get(workspace, datastore_name)
```

Use el `from_delimited_files()` método para leer archivos delimitados y crear un conjunto de datos no registrado.

```Python
# create an in-memory Dataset on your local machine
datapath = dstore.path('data/src/crime.csv')
dataset = Dataset.from_delimited_files(datapath)

# returns the first 5 rows of the Dataset as a pandas Dataframe.
dataset.head(5)
```

||ID|Case Number|Date|Bloquear|IUCR|Primary Type|DESCRIPCIÓN|Location Description|Arrest|Domestic|...|Ward|Community Area|FBI Code|X Coordinate|Y Coordinate|Year|Updated On|Latitud|Longitud|Ubicación|
|--|--|---|---|---|---|----|------|-------|------|-----|---|----|----|-----|-----|------|----|-----|----|----|-----
|0|10498554|HZ239907|4/4/2016 23:56|007XX E 111TH ST|1153|PRÁCTICA ENGAÑOSA|ROBO DE IDENTIDAD FINANCIERA A TRAVÉS DE 300 USD|OTHER|FALSE|FALSE|...|9|50|11|1183356|1831503|2016|5/11/2016 15:48|41.69283384|-87.60431945|(41.692833841, -87.60431945)|
1|10516598|HZ258664|4/15/2016 17:00|082XX S MARSHFIELD AVE|890|THEFT| DESDE LA CREACIÓN|RESIDENCIA|FALSE|FALSE|...|21|71|6|1166776|1850053|2016|5/12/2016 15:48|41.74410697|-87.66449429|(41.744106973, -87.664494285)
2|10519196|HZ261252|4/15/2016 10:00|104XX S SACRAMENTO AVE|1154|PRÁCTICA ENGAÑOSA|ROBO DE IDENTIDAD FINANCIERA 300 DÓLARES Y EN|RESIDENCIA|FALSE|FALSE|...|19|74|11|||2016|5/12/2016 15:50
3|10519591|HZ261534|4/15/2016 9:00|113XX UARDAR ASUSTADOS|1120|PRÁCTICA ENGAÑOSA|FALSIFICACIÓN|RESIDENCIA|FALSE|FALSE|...|9|49|10|||2016|5/13/2016 15:51
4|10534446|HZ277630|4/15/2016 10:00|055XX N KEDZIE AVE|890|THEFT|DESDE LA CREACIÓN|SCHOOL, PÚBLICO, CREAR|FALSE|FALSE|...|40|13|6|||2016|5/25/2016 15:59|

## <a name="register-your-datasets-with-workspace"></a>Registrar los conjuntos de datos con el área de trabajo

Use la [ `register()` ](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset.dataset?view=azure-ml-py#register-workspace--name--description-none--tags-none--visible-true--exist-ok-false--update-if-exist-false-) método para registrar los conjuntos de datos al área de trabajo para compartir y reutilizar dentro de su organización y entre varios experimentos.

```Python
dataset = dataset.register(workspace = workspace,
                           name = 'dataset_crime',
                           description = 'Training data',
                           exist_ok = False
                           )
```

>[!NOTE]
> El valor de parámetro predeterminado para `register()` es `exist_ok = False`. Si se intenta registrar un conjunto de datos con el mismo nombre sin cambiar esta configuración que se producirá un error.

El `register()` método devuelve el conjunto de datos ya registrado con el valor del parámetro, `exist_ok = True`.

```Python
dataset = dataset.register(workspace = workspace,
                           name = 'dataset_crime',
                           description = 'Training data',
                           exist_ok = True
                           )
```

Use `list()` para ver todos los conjuntos de datos registrados en el área de trabajo.

```Python
Dataset.list(workspace_name)
```

El código anterior genera lo siguiente:

```Python
[Dataset(Name: dataset_crime,
         Workspace: workspace_name)]
```

## <a name="access-datasets-in-workspace"></a>Conjuntos de datos de acceso de área de trabajo

Los conjuntos de datos registrados sean accesibles y consumibles localmente, remotamente y en clústeres de cálculo, como el proceso de Azure Machine Learning. Para reutilizar el conjunto de datos registrado en experimentos y entornos de proceso, utilice el código siguiente para obtener el área de trabajo y el conjunto de datos registrado por el nombre.

```Python
workspace = Workspace.from_config()

dataset = workspace.datasets['dataset_crime']
```

## <a name="next-steps"></a>Pasos siguientes

* [Explorar y preparar los conjuntos de datos](how-to-explore-prepare-data.md).
* [Administrar el ciclo de vida de las definiciones de conjunto de datos](how-to-manage-dataset-definitions.md).
* Para obtener un ejemplo del uso de conjuntos de datos, vea el [cuadernos de ejemplo](https://aka.ms/dataset-tutorial).
