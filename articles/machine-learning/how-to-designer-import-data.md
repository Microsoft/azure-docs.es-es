---
title: Importación de datos en el diseñador
titleSuffix: Azure Machine Learning
description: Aprenda a importar los datos en el diseñador de Azure Machine Learning desde varios orígenes de datos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: likebupt
ms.author: keli19
ms.date: 09/09/2020
ms.topic: conceptual
ms.custom: how-to, designer
ms.openlocfilehash: 15fca48327c46480546764be1b2ab40c1635e874
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90985604"
---
# <a name="import-data-into-azure-machine-learning-designer"></a>Importación de datos en el diseñador de Azure Machine Learning

En este artículo, aprenderá a importar sus propios datos en el diseñador para crear soluciones personalizadas. Hay dos formas de importar datos en el diseñador: 

* **Conjuntos de datos de Azure Machine Learning**: registre [conjuntos de datos](concept-data.md#datasets) en Azure Machine Learning para habilitar características avanzadas que le ayuden a administrar sus datos.
* **Módulo Import Data** (Importar datos): el módulo [Import data](algorithm-module-reference/import-data.md) (Importar datos) se usa para acceder directamente a datos de orígenes de datos en línea.

[!INCLUDE [machine-learning-missing-ui](../../includes/machine-learning-missing-ui.md)]

## <a name="use-azure-machine-learning-datasets"></a>Uso de conjuntos de datos de Azure Machine Learning

Se recomienda usar [conjuntos de datos](concept-data.md#datasets) para importar datos en el diseñador. Al registrar un conjunto de datos, puede aprovechar al máximo las características de datos avanzadas como el [control de versiones y el seguimiento](how-to-version-track-datasets.md), y la [supervisión de datos](how-to-monitor-datasets.md).

### <a name="register-a-dataset"></a>Registro de un conjunto de datos

Puede registrar los conjuntos de datos existentes [mediante programación con el SDK](how-to-create-register-datasets.md#datasets-sdk) o [visualmente en Azure Machine Learning Studio](how-to-connect-data-ui.md#create-datasets).

También puede registrar la salida de cualquier módulo del diseñador como un conjunto de datos.

1. Seleccione el módulo que genera los datos que desea registrar.

1. En el panel de propiedades, seleccione **Outputs + logs** (Salidas y registros) > **Register dataset** (Registrar conjunto de datos).

    ![Captura de pantalla que muestra cómo ir a la opción Register Dataset (Registrar conjunto de datos)](media/how-to-designer-import-data/register-dataset-designer.png)

Si los datos de salida del módulo están en formato tabular, debe optar por registrar la salida como un **conjunto de datos de archivo** o un **conjunto de datos tabular**.

 - El **conjunto de datos de archivo** registra la carpeta de salida del módulo como un conjunto de datos de archivo. La carpeta de salida contiene un archivo de datos y metarchivos que el diseñador usa internamente. Seleccione esta opción si quiere seguir usando el conjunto de los conjuntos registrado en el diseñador. 

 - El **conjunto de datos tabular** solo registra el archivo de datos de salida del módulo como un conjunto de datos tabular. Este formato se usa fácilmente en otras herramientas, como las de aprendizaje automático automatizado o el SDK de Python. Seleccione esta opción si tiene previsto utilizar el conjunto de datos registrado fuera del diseñador.  



### <a name="use-a-dataset"></a>Uso de un conjunto de datos

Sus conjuntos de datos registrados se pueden encontrar en la paleta del módulo, en **Conjuntos de datos**. Para usar un conjunto de datos, arrástrelo y suéltelo en el lienzo de la canalización. Luego, conecte el puerto de salida del conjunto de datos a otros módulos del lienzo. 

![Captura de pantalla que muestra la ubicación de los conjuntos de datos guardados en la paleta del diseñador](media/how-to-designer-import-data/use-datasets-designer.png)


> [!NOTE]
> El diseñador admite el [control de versiones del conjunto de datos](how-to-version-track-datasets.md). Especifique la versión del conjunto de datos en el panel de propiedades del módulo del conjunto de datos.


## <a name="import-data-using-the-import-data-module"></a>Importación de datos mediante el módulo Import Data (Importar datos)

Aunque es aconsejable usar conjuntos de datos para importar datos, también se puede usar el módulo [Import Data](algorithm-module-reference/import-data.md) (Importar datos). El módulo Import Data (Importar datos) omite el registro del conjunto de datos en Azure Machine Learning e importa los datos directamente de un [almacén de datos](concept-data.md#datastores) o la dirección URL HTTP.

Para más información acerca de cómo usar el módulo Import Data (Importar datos), consulte la [página de referencia Importar datos](algorithm-module-reference/import-data.md).

> [!NOTE]
> Si el conjunto de datos tiene demasiadas columnas, puede encontrar el siguiente error: "Validation failed due to size limitation" (Error de validación debido a un límite de tamaño). Para evitar esto, [registre el conjunto de datos en la interfaz de conjuntos de datos](how-to-connect-data-ui.md#create-datasets).

## <a name="supported-sources"></a>Orígenes compatibles

En esta sección se enumeran los orígenes de datos que admite el diseñador. Los datos llegan al diseñador desde un almacén de datos o un [conjunto de datos tabular](how-to-create-register-datasets.md#dataset-types).

### <a name="datastore-sources"></a>Orígenes del almacén de datos
Para ver una lista de los orígenes del almacén de datos compatibles, consulte [Acceso a los datos en los servicios de almacenamiento de Azure](how-to-access-data.md#supported-data-storage-service-types).

### <a name="tabular-dataset-sources"></a>Orígenes de conjuntos de datos tabulares

El diseñador admite los conjuntos de datos tabulares creados en los siguientes orígenes:
 * Archivos delimitados
 * Archivos JSON
 * Archivos de Parquet
 * Consultas SQL

## <a name="data-types"></a>Tipos de datos

Internamente, el diseñador reconoce los siguientes tipos de datos:

* String
* Entero
* Decimal
* Boolean
* Date

El diseñador usa un tipo de datos interno para pasar datos entre los módulos. Puede convertir explícitamente sus datos en formato de tabla de datos con el módulo [Convertir al conjunto de datos](algorithm-module-reference/convert-to-dataset.md). Todos los módulos que acepten formatos que no sean el interno convertirán los datos de manera silenciosa antes de pasarlos al módulo siguiente.

## <a name="data-constraints"></a>Restricciones de datos

Los módulos del diseñador están limitados por el tamaño del destino de proceso. Para conjuntos de datos mayores, debe usar un recurso de proceso de Azure Machine Learning mayor. Para más información sobre el proceso de Azure Machine Learning, consulte [¿Qué son los destinos de proceso en Azure Machine Learning?](concept-compute-target.md#azure-machine-learning-compute-managed)

## <a name="access-data-in-a-virtual-network"></a>Acceso a los datos de una red virtual

Si el área de trabajo está en una red virtual, debe realizar pasos de configuración adicionales para visualizar los datos en el diseñador. Para más información sobre cómo usar los almacenes de datos y los conjuntos de datos en una red virtual, consulte [Uso de Azure Machine Learning Studio en una instancia de Azure Virtual Network](how-to-enable-studio-virtual-network.md).

## <a name="next-steps"></a>Pasos siguientes

Conozca los aspectos básicos del diseñador con el [Tutorial: Predecir el precio del automóvil con el diseñador](tutorial-designer-automobile-price-train-score.md).
