---
title: Transformación de datos en el diseñador (versión preliminar)
titleSuffix: Azure Machine Learning
description: Aprenda a transformar datos en el diseñador de Azure Machine Learning (versión preliminar) para crear sus propios conjuntos de datos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: peterclu
ms.author: peterlu
ms.date: 06/28/2020
ms.topic: conceptual
ms.custom: how-to, designer
ms.openlocfilehash: 05a21ce10db2822c963f1b375842e9a7233e0816
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87457828"
---
# <a name="transform-data-in-azure-machine-learning-designer-preview"></a>Transformación de datos en el diseñador de Azure Machine Learning (versión preliminar)
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-enterprise-sku.md)]

En este artículo, aprenderá a transformar y guardar conjuntos de datos en el diseñador de Azure Machine Learning para que pueda preparar sus propios datos para el aprendizaje automático.

Usará el conjunto de datos [Clasificación binaria de ingresos en el censo de adultos](sample-designer-datasets.md) de ejemplo para preparar dos conjuntos de datos: uno que incluya la información del censo de datos solo de los Estados Unidos y otro que incluya la información del censo correspondiente a adultos que no son de EE. UU.

En este artículo aprenderá a:

1. Transformar un conjunto de datos para prepararlo para el entrenamiento.
1. Exportar los conjuntos de datos resultantes a un almacén de datos.
1. Ver los resultados.

Este procedimiento es requisito previo para el artículo sobre cómo [volver a entrenar modelos con el diseñador](how-to-retrain-designer.md). En ese artículo, aprenderá a usar los conjuntos de datos transformados para entrenar varios modelos con parámetros de canalización.

[!INCLUDE [machine-learning-missing-ui](../../includes/machine-learning-missing-ui.md)]

## <a name="transform-a-dataset"></a>Transformación de un conjunto de datos

En esta sección, aprenderá a importar el conjunto de datos de ejemplo y a dividirlos en conjuntos de datos de EE. UU. y no de EE. UU. Para más información sobre cómo importar sus propios datos al diseñador, consulte [cómo importar los datos](how-to-designer-import-data.md).

### <a name="import-data"></a>Importar datos

Siga estos pasos para importar el conjunto de datos de ejemplo.

1. Inicie sesión en <a href="https://ml.azure.com?tabs=jre" target="_blank">ml.azure.com</a> y seleccione el área de trabajo con la que quiere trabajar.

1. Vaya al diseñador. Seleccione **Easy-to-use-prebuild modules** (Módulos fáciles de usar creados previamente) para crear una canalización.

1. Seleccione un destino de proceso predeterminado para ejecutar la canalización.

1. A la izquierda del lienzo de la canalización, hay una paleta de conjuntos de datos y módulos. Seleccione **Conjuntos de datos**. Luego, consulte la sección **Ejemplos**.

1. Arrastre y coloque el conjunto de datos **Clasificación binaria de ingresos en el censo de adultos** en el lienzo.

1. Seleccione el módulo del conjunto de datos **Ingresos en el censo de adultos**.

1. En el panel de detalles que aparece a la derecha del lienzo, seleccione **Outputs** (Resultados).

1. Seleccione el icono de visualización. ![icono de visualización](media/how-to-designer-transform-data/visualize-icon.png).

1. Use la ventana de vista previa de los datos para explorar el conjunto de datos. Tenga en cuenta especialmente los valores de la columna "native-country".

### <a name="split-the-data"></a>División de los datos

En esta sección, usará el [módulo Split Data](algorithm-module-reference/split-data.md) (Dividir datos) para identificar y dividir las filas que contienen "United-States" en la columna "native-country". 

1. En la paleta de módulos que se encuentra a la izquierda del lienzo, expanda la sección **Transformación de datos** y busque el módulo **Split Data** (Dividir datos).

1. Arrastre el módulo **Split Data** (Dividir datos) al lienzo y colóquelo debajo del módulo del conjunto de datos.

1. Conecte el módulo del conjunto de datos con el módulo **Split Data** (Dividir datos).

1. Seleccione el módulo **Split Data** (Dividir datos).

1. En el panel de detalles del módulo que se encuentra a la derecha del lienzo, establezca **Splitting mode** (Modo de división) en **Expresión regular**.

1. Escriba la **expresión regular**: `\"native-country" United-States`.

    El modo **Expresión regular** prueba una columna con respecto a un valor. Para más información sobre el módulo Split Data (Dividir datos), consulte la [página de referencia del módulo de algoritmo](algorithm-module-reference/split-data.md).

La canalización debe ser parecida a esta:

:::image type="content" source="./media/how-to-designer-transform-data/split-data.png"alt-text="Captura de pantalla que muestra cómo configurar la canalización y el módulo Split Data (Dividir datos)":::


## <a name="save-the-datasets"></a>Guardado de los conjuntos de datos

Ahora que la canalización está configurada para dividir los datos, debe especificar dónde quiere guardar los conjuntos de datos. En este ejemplo, use el módulo **Export Data** (Exportar datos) para guardar los datos en un almacén de datos. Para más información sobre los almacenes de datos, consulte [Conexión con los servicios de Azure Storage](how-to-access-data.md).

1. En la paleta de módulos que se encuentra a la izquierda del lienzo, expanda la sección **Entrada y salida de datos** y busque el módulo **Export Data** (Exportar datos).

1. Arrastre y coloque dos módulos **Export Data** (Exportar datos) debajo del módulo **Split Data** (Dividir datos).

1. Conecte cada puerto de salida del módulo **Split Data** (Dividir datos) a un módulo **Export Data** (Exportar datos) distinto.

    La canalización debe parecerse a esta:

    ![Captura de pantalla que muestra cómo conectar los módulos Export Data (Exportar datos)](media/how-to-designer-transform-data/export-data-pipeline.png).

1. Seleccione el módulo **Export Data** (Exportar datos) que está conectado al puerto que está más a la *izquierda* del módulo **Split Data** (Dividir datos).

    El orden de los puertos de salida es importante para el módulo **Split Data** (Dividir datos). El primer puerto de salida contiene las filas en las que el valor de la expresión regular es true. En este caso, el primer puerto contiene filas para los ingresos basados en Estados Unidos y el segundo puerto contiene filas para ingresos no basados en Estados Unidos.

1. En el panel de detalles del módulo que se encuentra a la derecha del lienzo, establezca estas opciones:
    
    **Tipo de almacén de datos**: Azure Blob Storage

    **Almacén de datos**: Seleccione un almacén de datos existente o seleccione "Nuevo almacén de datos" para crear uno ahora.

    **Ruta de acceso**: `/data/us-income`

    **Formato de archivo**: csv

    > [!NOTE]
    > En este artículo se da por supuesto que tiene acceso a un almacén de datos registrado en el área de trabajo de Azure Machine Learning actual. Para instrucciones sobre cómo configurar un almacén de datos, consulte [Conexión con los servicios de Azure Storage](how-to-access-data.md#studio).

    Si no tiene un almacén de datos, puede crear uno ahora. Para fines de ejemplo, en este artículo se guardarán los conjuntos de datos en la cuenta de almacenamiento de blobs predeterminada asociada con el área de trabajo. Guardará los conjuntos de datos en el contenedor `azureml` en una carpeta nueva llamada `data`.

1.  Seleccione el módulo **Export Data** (Exportar datos) conectado al puerto que está más a la *derecha* del módulo **Split Data** (Dividir datos).

1. En el panel de detalles del módulo que se encuentra a la derecha del lienzo, establezca estas opciones:
    
    **Tipo de almacén de datos**: Azure Blob Storage

    **Almacén de datos**: seleccione el mismo almacén de datos que antes

    **Ruta de acceso**: `/data/non-us-income`

    **Formato de archivo**: csv

1. Confirme que el módulo **Export Data** (Exportar datos) conectado al puerto izquierdo del módulo **Split Data** (Dividir datos) tiene la **ruta de acceso** `/data/us-income`.

1. Confirme que el módulo **Export Data** (Exportar datos) conectado al puerto derecho tiene la **ruta de acceso** `/data/non-us-income`.

    La canalización y configuración deberían ser similares a estas:
    
    ![Captura de pantalla que muestra cómo configurar los módulos Export Data (Exportar datos)](media/how-to-designer-transform-data/us-income-export-data.png).

### <a name="submit-the-run"></a>Envío de la ejecución

Ahora que la canalización está configurada para dividir y exportar los datos, envíe la ejecución de una canalización.

1. En la parte superior del lienzo, seleccione **Enviar**.

1. En el cuadro de diálogo **Configurar ejecución de canalización**, seleccione **Crear nuevo** para crear un experimento.

    Los experimentos agrupan de manera lógica las ejecuciones de canalizaciones relacionadas. Si ejecuta esta canalización en el futuro, debe usar el mismo experimento con fines de registro y seguimiento.

1. Proporcione un nombre descriptivo para el experimento como "split-census-data".

1. Seleccione **Submit** (Enviar).

## <a name="view-results"></a>Vista de resultados

Una vez finalizada la ejecución de la canalización, puede ver los resultados si navega al almacenamiento de blobs en Azure Portal. También puede ver los resultados intermedios del módulo **Split Data** (Dividir datos) para confirmar que los datos se dividieron correctamente.

1. Seleccione el módulo **Split Data** (Dividir datos).

1. En el panel de detalles del módulo, situado a la derecha del lienzo, seleccione **Outputs + logs** (Salidas y registros). 

1. Seleccione el icono de visualización ![icono de visualización](media/how-to-designer-transform-data/visualize-icon.png) junto a **Conjunto de datos de resultados 1**. 

1. Compruebe que la columna "native-country" solo contiene el valor "United-States".

1. Seleccione el icono de visualización ![icono de visualización](media/how-to-designer-transform-data/visualize-icon.png) junto a **Conjunto de datos de resultados 2**. 

1. Compruebe que la columna "native-country" no contiene el valor "United-States".

## <a name="clean-up-resources"></a>Limpieza de recursos

Omita esta sección si quiere seguir con la segunda parte de este procedimiento, [Volver a entrenar modelos con el diseñador de Azure Machine Learning](how-to-retrain-designer.md).

[!INCLUDE [aml-ui-cleanup](../../includes/aml-ui-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este artículo aprendió a transformar un conjunto de datos y guardarlo en un almacén de datos registrado.

Vaya a la próxima parte de esta serie de procedimientos con [Volver a entrenar modelos con el diseñador de Azure Machine Learning](how-to-retrain-designer.md) para usar los conjuntos de datos transformados y los parámetros de canalización para entrenar los modelos de aprendizaje automático.
