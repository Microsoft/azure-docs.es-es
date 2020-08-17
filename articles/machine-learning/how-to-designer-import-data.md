---
title: Importación de datos en el diseñador (versión preliminar)
titleSuffix: Azure Machine Learning
description: Aprenda a importar los datos en el diseñador de Azure Machine Learning (versión preliminar) desde varios orígenes de datos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: peterclu
ms.author: peterlu
ms.date: 01/16/2020
ms.topic: conceptual
ms.custom: how-to, designer
ms.openlocfilehash: d977c8e13ce75eb276c8fdb11e9dd40e40a923ad
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495378"
---
# <a name="import-data-into-azure-machine-learning-designer-preview"></a>Importación de datos en el diseñador de Azure Machine Learning (versión preliminar)

En este artículo, aprenderá a importar sus propios datos en el diseñador para crear soluciones personalizadas. Hay dos formas de importar datos en el diseñador: 

* **Conjuntos de datos de Azure Machine Learning**: registre [conjuntos de datos](concept-data.md#datasets) en Azure Machine Learning para habilitar características avanzadas que le ayuden a administrar sus datos.
* **Módulo Import Data** (Importar datos): el módulo [Import data](algorithm-module-reference/import-data.md) (Importar datos) se usa para acceder directamente a datos de orígenes de datos en línea.

[!INCLUDE [machine-learning-missing-ui](../../includes/machine-learning-missing-ui.md)]

## <a name="use-azure-machine-learning-datasets"></a>Uso de conjuntos de datos de Azure Machine Learning

Se recomienda usar [conjuntos de datos](concept-data.md#datasets) para importar datos en el diseñador. Al registrar un conjunto de datos, puede aprovechar al máximo las características de datos avanzadas como el [control de versiones y el seguimiento](how-to-version-track-datasets.md), y la [supervisión de datos](how-to-monitor-datasets.md).

### <a name="register-a-dataset"></a>Registro de un conjunto de datos

Puede registrar los conjuntos de datos existentes [mediante programación con el SDK](how-to-create-register-datasets.md#datasets-sdk) o [visualmente en Azure Machine Learning Studio](how-to-create-register-datasets.md#datasets-ui).

También puede registrar la salida de cualquier módulo del diseñador como un conjunto de datos.

1. Seleccione el módulo que genera los datos que desea registrar.

1. En el panel de propiedades, seleccione **Salidas** > **Visualizar**.

    ![Captura de pantalla que muestra cómo ir a la opción Register Dataset (Registrar conjunto de datos)](media/how-to-designer-import-data/register-dataset-designer.png)

### <a name="use-a-dataset"></a>Uso de un conjunto de datos

Sus conjuntos de datos registrados se pueden encontrar en la paleta del módulo, en **Datasets** > **My Datasets** (Conjuntos de datos > Mis conjuntos de datos). Para usar un conjunto de datos, arrástrelo y suéltelo en el lienzo de la canalización. Luego, conecte el puerto de salida del conjunto de datos a otros módulos de la paleta.

![Captura de pantalla que muestra la ubicación de los conjuntos de datos guardados en la paleta del diseñador](media/how-to-designer-import-data/use-datasets-designer.png)


> [!NOTE]
> Actualmente, el diseñador solo admite el procesamiento de [conjuntos de datos tabulares](how-to-create-register-datasets.md#dataset-types). Si desea usar [conjuntos de datos de archivo](how-to-create-register-datasets.md#dataset-types), use el SDK de Azure Machine Learning disponible para Python y R.

## <a name="import-data-using-the-import-data-module"></a>Importación de datos mediante el módulo Import Data (Importar datos)

Aunque es aconsejable usar conjuntos de datos para importar datos, también se puede usar el módulo [Import Data](algorithm-module-reference/import-data.md) (Importar datos). El módulo Import Data (Importar datos) omite el registro del conjunto de datos en Azure Machine Learning e importa los datos directamente de un [almacén de datos](concept-data.md#datastores) o la dirección URL HTTP.

Para más información acerca de cómo usar el módulo Import Data (Importar datos), consulte la [página de referencia Importar datos](algorithm-module-reference/import-data.md).

> [!NOTE]
> Si el conjunto de datos tiene demasiadas columnas, puede encontrar el siguiente error: "Validation failed due to size limitation" (Error de validación debido a un límite de tamaño). Para evitar esto, [registre el conjunto de datos en la interfaz de conjuntos de datos](how-to-create-register-datasets.md#datasets-ui).

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

Si el área de trabajo está en una red virtual, debe realizar pasos de configuración adicionales para visualizar los datos en el diseñador. Para más información sobre cómo usar los almacenes de datos y los conjuntos de datos en una red virtual, consulte [Aislamiento de red durante el entrenamiento e inferencia con redes virtuales privadas](how-to-enable-virtual-network.md#machine-learning-studio).

## <a name="next-steps"></a>Pasos siguientes

Aprenda los conceptos básicos del diseñador con [Tutorial: Predecir el precio del automóvil con el diseñador](tutorial-designer-automobile-price-train-score.md).
