---
title: Análisis de datos con Azure Machine Learning
description: Use Azure Machine Learning para crear un modelo de aprendizaje automático predictivo basado en los datos almacenados en Azure Synapse.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: machine-learning
ms.date: 07/15/2020
ms.author: martinle
ms.reviewer: igorstan
ms.custom: seo-lt-2019
tag: azure-Synapse
ms.openlocfilehash: 9cf65b2fdeb7faa03b950593db86dd32a4ef91a7
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86495745"
---
# <a name="analyze-data-with-azure-machine-learning"></a>Análisis de datos con Azure Machine Learning

En este tutorial se usa [Azure Machine Learning Designer](https://docs.microsoft.com/azure/machine-learning/concept-designer) para crear un modelo de aprendizaje automático predictivo. El modelo se basa en los datos almacenados en Azure Synapse. El escenario del tutorial es predecir la probabilidad de que un cliente compre una bicicleta y su finalidad es que Adventure Works, la tienda de bicicletas, pueda crear una campaña de marketing dirigida.

## <a name="prerequisites"></a>Requisitos previos

Para realizar este tutorial, necesita:

* Un grupo de SQL con la base de datos de ejemplo AdventureWorksDW previamente cargada. Para aprovisionar este grupo de SQL, consulte [Creación de un grupo de SQL](create-data-warehouse-portal.md) y cargue los datos de ejemplo. Si ya tiene un almacenamiento de datos pero no tiene datos de ejemplo, puede [cargar manualmente los datos de ejemplo](load-data-from-azure-blob-storage-using-polybase.md).
* Un área de trabajo de Azure Machine Learning. Siga [este tutorial](https://docs.microsoft.com/azure/machine-learning/how-to-manage-workspace) para crear una nueva.

## <a name="get-the-data"></a>Obtener los datos

Los datos usados están en la vista dbo.vTargetMail en AdventureWorksDW. Para usar el almacén de datos en este tutorial, los datos se exportan primero a la cuenta de Azure Data Lake Storage, ya que en la actualidad Azure Synapse no admite conjuntos de datos. Azure Data Factory se puede usar para exportar datos desde el almacenamiento de datos a Azure Data Lake Storage mediante la [actividad de copia](https://docs.microsoft.com/azure/data-factory/copy-activity-overview). Use la siguiente consulta para la importación:

```sql
SELECT [CustomerKey]
  ,[GeographyKey]
  ,[CustomerAlternateKey]
  ,[MaritalStatus]
  ,[Gender]
  ,cast ([YearlyIncome] as int) as SalaryYear
  ,[TotalChildren]
  ,[NumberChildrenAtHome]
  ,[EnglishEducation]
  ,[EnglishOccupation]
  ,[HouseOwnerFlag]
  ,[NumberCarsOwned]
  ,[CommuteDistance]
  ,[Region]
  ,[Age]
  ,[BikeBuyer]
FROM [dbo].[vTargetMail]
```

Una vez que los datos están disponibles en Azure Data Lake Storage, los almacenes de datos de Azure Machine Learning se utilizan para [conectarse a los servicios de almacenamiento de Azure](https://docs.microsoft.com/azure/machine-learning/how-to-access-data). Siga los pasos que se indican a continuación para crear un almacén de datos y el correspondiente conjunto de datos:

1. Inicie Azure Machine Learning Studio desde Azure Portal o inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/).

1. Haga clic en **Almacenes de datos** en el panel izquierdo de la sección **Administrar** y, a continuación, haga clic en **New Datastore** (Nuevo almacén de datos).

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/datastores-tab.png" alt-text="Captura de pantalla del panel izquierdo de la interfaz de Azure Machine Learning":::

1. Proporcione un nombre para el almacén de datos, seleccione el tipo como "Azure Blob Storage", facilite la ubicación y las credenciales. A continuación, haga clic en **Crear**.

1. A continuación, haga clic en **Conjuntos de datos** en el panel izquierdo en la sección **Recursos**. Seleccione **Crear conjunto de datos** con la opción **De almacén de datos**.

1. Especifique el nombre del conjunto de datos y seleccione el tipo para que sea **Tabular**. A continuación, haga clic en **Siguiente** para avanzar.

1. En **Select or create a datastore section** (Seleccionar o crear una sección de almacén de datos), seleccione la opción **Previously created datastore** (Almacén de datos creado anteriormente). Seleccione el almacén de datos que se creó anteriormente. Haga clic en Siguiente y especifique la ruta de acceso y la configuración del archivo. Asegúrese de especificar el encabezado de columna, si los archivos contienen uno.

1. Por último, haga clic en **Crear** para crear el conjunto de datos.

## <a name="configure-designer-experiment"></a>Configuración del experimento del diseñador

Después, siga los pasos que aparecen a continuación para configurar el diseñador:

1. Haga clic en la pestaña **Designer** en el panel izquierdo en la sección **Autor**.

1. Seleccione **Módulos creados previamente fáciles de usar** para crear una nueva canalización.

1. En el panel de valores de la derecha, especifique el nombre de la canalización.

1. Seleccione también un clúster aprovisionado previamente como clúster de proceso de destino para todo el experimento. Cierre la ventana Configuración.

## <a name="import-the-data"></a>Importación de los datos

1. Seleccione la subpestaña **Conjuntos de datos** en el panel izquierdo, debajo del cuadro de búsqueda.

1. Arrastre el conjunto de datos que creó anteriormente al lienzo.

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/import-dataset.png" alt-text="Captura de pantalla del módulo de conjunto de datos en el lienzo.":::

## <a name="clean-the-data"></a>Limpiar los datos

Para limpiar los datos, anule las columnas que no sean pertinentes para el modelo. Para hacerlo, siga estos pasos:

1. Seleccione la subpestaña **Módulos** en el panel izquierdo.

1. Arrastre el módulo **Seleccionar columnas en el conjunto de datos** bajo la opción **Transformación de datos < Manipulación** en el lienzo. Conecte este módulo con el módulo del **Conjunto datos**.

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/select-columns-zoomed-in.png" alt-text="Captura de pantalla del módulo de selección de columnas en el lienzo." lightbox="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/select-columns-zoomed-out.png":::

1. Haga clic en el módulo para abrir el panel Propiedades. Haga clic en Editar columna para especificar cuáles son las columnas que desea anular.

1. Excluya dos columnas: CustomerAlternateKey y GeographyKey. Haga clic en **Guardar**

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/drop-columns.png" alt-text="Captura de pantalla con las columnas que se van a anular.":::

## <a name="build-the-model"></a>Generar el modelo

Los datos se dividen en 80-20: 80 % para entrenar un modelo de aprendizaje automático y un 20 % para probar el modelo. En este problema de clasificación binaria se usan los algoritmos de "dos clases".

1. Arrastre el módulo **Split Data** (Dividir datos) hasta el lienzo.

1. En el panel de propiedades, escriba 0,8 en **Fraction of rows in the first output dataset** (Fracción de filas del primer conjunto de datos de salida).

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/split-data.png" alt-text="Captura de pantalla que muestra la relación de división de 0,8.":::

1. Arrastre el módulo **Árbol de decisión aumentado de dos clases** al lienzo.

1. Arrastre el módulo **Train Model** (Entrenar modelo) hasta el lienzo. Especifique las entradas conectándolo a los módulos **Two-Class Boosted Decision Tree** (Árbol de decisión promovido por dos clases), que es un algoritmo de Machine Learning, y **Split Data** (Dividir datos), que son los datos para entrenar el algoritmo.

1. Para el módulo Train Model (Entrenar modelo) en la opción **Label column** (Columna de etiqueta) del panel Propiedades, seleccione Editar columna. Seleccione la columna **BikeBuyer** como columna de predicción y haga clic en **Guardar**.

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/label-column.png" alt-text="Captura de pantalla que muestra la columna de etiqueta, BikeBuyer, seleccionada.":::

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/train-model.png" alt-text="Captura de pantalla del módulo Train Model (Entrenar modelo) conectado a los módulos Two-Class Boosted Decision Tree (Árbol de decisión promovido por dos clases) y Split Data (Dividir datos).":::

## <a name="score-the-model"></a>Puntuación del modelo

Ahora pruebe cómo funciona el modelo con datos de prueba. Se compararán dos algoritmos distintos para ver cuál funciona mejor. Para hacerlo, siga estos pasos:

1. Arrastre el módulo **Modelo de puntuación** al lienzo y conéctelo a los módulos **Modelo de entrenamiento** y **Datos divididos**.

1. Arrastre **Two-Class Bayes Averaged Perceptron** (Perceptrón promedio de Bayes de dos clases) al lienzo del experimento. Podrá ver cómo funciona este algoritmo en comparación con Two-Class Boosted Decision Tree (Árbol de decisión promovido por dos clases).

1. Copie y pegue los módulos **Train Model** (Entrenar modelo) y **Score Model** (Puntuar modelo) en el lienzo.

1. Arrastre el módulo **Evaluar modelo** al lienzo para comparar los dos algoritmos.

1. Haga clic en **Enviar** para configurar la ejecución de la canalización.

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/algo-comparison-zoomed-in.png" alt-text="Captura de pantalla de todos los módulos restantes en el lienzo." lightbox="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/algo-comparison-zoomed-out.png":::

1. Una vez finalizada la ejecución, haga clic con el botón derecho en el módulo **Evaluate Model** (Evaluar modelo) y haga clic en **Visualize Evaluation results** (Visualizar resultados de evaluación).

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/result-visualize-zoomed-out.png" alt-text="Captura de pantalla de los resultados.":::

Las métricas proporcionadas son la curva ROC, el diagrama de retirada-precisión y la curva de elevación. Al mirar estas métricas, se puede observar que el primer modelo funciona mejor que el segundo. Para ver lo que predijo el primer modelo, haga clic con el botón derecho en el módulo Score Model (Puntuar modelo) y haga clic en Visualize Scored results (Visualizar resultados con puntuación) para ver los resultados previstos.

Verá dos columnas más agregadas al conjunto de datos de prueba.

* Probabilidades puntuadas: la probabilidad de que un cliente sea comprador de bicicletas.
* Etiquetas puntuadas: la clasificación realizada por el modelo – comprador de bicicletas (1) o no (0). Este umbral de probabilidad para etiquetar se establece en 50% y se puede ajustar.

Para ver cómo ha funcionado el modelo compare la columna BikeBuyer (real) con la columna de predicción Scored Labels (Etiquetas puntuadas). A continuación, puede utilizar este modelo para hacer predicciones para nuevos clientes. Puede [publicar este modelo como un servicio web](https://docs.microsoft.com/azure/machine-learning/tutorial-designer-automobile-price-deploy) o escribir los resultados en Azure Synapse.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Machine Learning, consulte [Introducción a Machine Learning en Azure](https://docs.microsoft.com/azure/machine-learning/overview-what-is-azure-ml).

Obtenga [aquí](/sql/t-sql/queries/predict-transact-sql?view=azure-sqldw-latest) información acerca de la puntuación integrada en el almacenamiento de datos.