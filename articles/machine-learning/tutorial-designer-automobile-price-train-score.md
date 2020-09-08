---
title: 'Tutorial: Predicción de los precios de los automóviles con el diseñador (versión preliminar)'
titleSuffix: Azure Machine Learning
description: Aprenda a entrenar, puntuar e implementar un modelo de Machine Learning mediante una interfaz de arrastrar y colocar. Este tutorial es la primera de una serie de dos partes sobre la predicción de precios de automóviles mediante la regresión lineal.
author: peterclu
ms.author: peterlu
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.date: 06/28/2020
ms.custom: designer
ms.openlocfilehash: 3992bbe466241eb388874905237dbe00f07b3752
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88812992"
---
# <a name="tutorial-predict-automobile-price-with-the-designer-preview"></a>Tutorial: Predicción del precio de un automóvil con el diseñador (versión preliminar)
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-enterprise-sku.md)]

En este tutorial de dos partes aprenderá a usar el diseñador de Azure Machine Learning para entrenar e implementar un modelo de Machine Learning que predice el precio de cualquier automóvil. El diseñador es una herramienta de arrastrar y colocar que le permite crear modelos de Machine Learning sin una sola línea de código.

En la primera parte del tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una canalización.
> * Importar datos.
> * Preparar los datos.
> * Entrenar un modelo de Machine Learning.
> * Evaluar un modelo de Machine Learning.

En la [segunda parte](tutorial-designer-automobile-price-deploy.md) del tutorial implementará el modelo como punto de conexión de inferencia en tiempo real para predecir el precio de cualquier automóvil en función de las especificaciones técnicas que envíe. 

> [!NOTE]
>Está disponible una versión completa de este tutorial como una canalización de ejemplo.
>
>Para encontrarla, vaya al diseñador del área de trabajo. En la sección **Nueva canalización**, seleccione **Sample 1 - Regression: Automobile Price Prediction (Basic)** (Ejemplo 1 - Regresión: predicción del precio de automóviles [básica])

[!INCLUDE [machine-learning-missing-ui](../../includes/machine-learning-missing-ui.md)]

## <a name="create-a-new-pipeline"></a>Creación de una canalización

Las canalizaciones de Azure Machine Learning organizan varios pasos de aprendizaje automático y procesamiento de datos en un único recurso. Las canalizaciones permiten organizar, administrar y reutilizar flujos de trabajo de aprendizaje automático complejos entre proyectos y usuarios.

Para crear una canalización de Azure Machine Learning, necesita un área de trabajo de Azure Machine Learning. En esta sección aprenderá a crear estos dos recursos.

### <a name="create-a-new-workspace"></a>Crear un área de trabajo

Para usar el diseñador, primero necesita un área de trabajo de Azure Machine Learning. El área de trabajo es el recurso de nivel superior de Azure Machine Learning y proporciona un lugar centralizado para trabajar con todos los artefactos que crea con Azure Machine Learning.

Si tiene un área de trabajo de Azure Machine Learning con una edición Enterprise, [vaya a la siguiente sección](#create-the-pipeline).

[!INCLUDE [aml-create-portal](../../includes/aml-create-in-portal-enterprise.md)]

### <a name="create-the-pipeline"></a>Creación de la canalización

1. Inicie sesión en <a href="https://ml.azure.com?tabs=jre" target="_blank">ml.azure.com</a> y seleccione el área de trabajo con la que quiere trabajar.

1. Seleccione **Diseñador**.

    ![Captura de pantalla del área de trabajo visual que muestra cómo acceder al diseñador](./media/tutorial-designer-automobile-price-train-score/launch-designer.png)

1. Seleccione **Easy-to-use prebuilt modules** (Módulos precompilados fáciles de usar).

1. En la parte superior del lienzo, seleccione el nombre predeterminado de la canalización, **Pipeline-Created-on** (Canalización creada el). Cambie el nombre a *Automobile price prediction* (Predicción del precio de automóviles). No es necesario que el nombre sea único.

## <a name="set-the-default-compute-target"></a>Configuración del destino de proceso predeterminado

Una canalización se ejecuta en un destino de proceso que es un recurso de proceso asociado al área de trabajo. Después de crear un destino de proceso, puede volver a usarlo para futuras ejecuciones.

Puede establecer un **destino de proceso predeterminado** para toda la canalización. Este indicará a cada módulo que debe usar el mismo destino de proceso de forma predeterminada. Sin embargo, puede especificar destinos de proceso por módulo.

1. Junto al nombre de la canalización, seleccione el **icono de engranaje** ![Captura de pantalla del icono de engranaje](./media/tutorial-designer-automobile-price-train-score/gear-icon.png) en la parte superior del lienzo para abrir el panel de **configuración**.

1. En el panel de **configuración** situado a la derecha del lienzo, elija **Selección de destino de proceso**.

    Si ya tiene un destino de proceso disponible, puede seleccionarlo para ejecutar esta canalización.

    > [!NOTE]
    > El diseñador solo puede ejecutar experimentos de entrenamiento en Azure Machine Learning Compute e instancias de proceso de Azure Machine Learning, pero no se mostrarán otros destinos de proceso.

1. Escriba un nombre para el recurso de proceso.

1. Seleccione **Guardar**.

    > [!NOTE]
    > La creación de un recurso de proceso tarda aproximadamente cinco minutos. Una vez creado el recurso, puede volver a usarlo y eliminar este tiempo de espera en futuras ejecuciones.
    >
    > El recurso de proceso se escala automáticamente a cero nodos cuando está inactivo para ahorrar costes. Cuando se vuelve a usar después de un retraso, es posible que experimente aproximadamente cinco minutos de tiempo de espera mientras se escala la copia de seguridad.

## <a name="import-data"></a>Importar datos

Se incluyen varios conjuntos de datos de ejemplo en el diseñador para que experimente con ellos. En este tutorial, use **Automobile price data (Raw)** . 

1. A la izquierda del lienzo de la canalización, hay una paleta de conjuntos de datos y módulos. Seleccione **Datasets** (Conjuntos de datos) y examine la sección **Samples** (Ejemplos) para ver los conjuntos de datos de ejemplo disponibles.

1. Seleccione el conjunto de datos **Automobile price data (Raw)** y arrástrelo al lienzo.

   ![Arrastrar datos al lienzo](./media/tutorial-designer-automobile-price-train-score/drag-data.gif)

### <a name="visualize-the-data"></a>Visualización de los datos

Puede visualizar los datos para comprender el conjunto de datos que va a usar.

1. Haga clic con el botón derecho en **Automobile price data (Raw)** (Datos de precio del automóvil [sin procesar]) y seleccione **Visualizar**.

1. Seleccione las diferentes columnas de la ventana de datos para ver información sobre cada una.

    Cada fila representa un automóvil y las variables asociadas a cada uno aparecen como columnas. Hay 205 filas y 26 columnas en este conjunto de datos.

## <a name="prepare-data"></a>Preparación de los datos

Los conjuntos de datos suelen necesitar algún procesamiento previo antes del análisis. Puede que, al inspeccionar el conjunto de datos, haya observado que faltan algunos valores. Estos valores que faltan se deben limpiar, para que el modelo pueda analizar los datos de manera adecuada.

### <a name="remove-a-column"></a>Quitar una columna

Al entrenar un modelo, hay que hacer algo con los datos que faltan. En este conjunto de datos, faltan muchos valores en la columna **normalized-losses**, por lo que excluiremos toda esa columna del modelo.

1. En la paleta de módulos que se encuentra a la izquierda del lienzo, expanda la sección **Transformación de datos** y busque el módulo **Select Columns in Dataset** (Seleccionar columnas del conjunto de datos).

1. Haga clic y arrastre el módulo **Select Columns in Dataset** (Seleccionar columnas del conjunto de datos) al lienzo. Coloque el módulo bajo el módulo del conjunto de datos.

1. Conecte el conjunto de datos **Automobile price data (Raw)** al módulo **Select Columns in Dataset** (Seleccionar columnas del conjunto de datos). Arrastre desde el puerto de salida del conjunto de datos, que es el círculo pequeño situado en la parte inferior del conjunto de datos en el lienzo, hasta el puerto de entrada de **Select Columns in Dataset** (Seleccionar columnas del conjunto de datos), que es el círculo pequeño de la parte superior del módulo.

    > [!TIP]
    > Puede crear un flujo de datos mediante la canalización si conecta el puerto de salida de un módulo al puerto de entrada de otro.
    >

    ![Conectar módulos](./media/tutorial-designer-automobile-price-train-score/connect-modules.gif)

1. Seleccione el módulo **Select Columns in Dataset**.

1. En el panel de detalles del módulo, situado a la derecha del lienzo, seleccione **Editar columna**.

1. Expanda la lista desplegable **Nombres de columna** situada junto a **Incluir** y seleccione **Todas las columnas**.

1. Seleccione el signo **+** para agregar una nueva regla.

1. En los menús desplegables, seleccione **Excluir** y **Nombres de columna**.
    
1. Escriba *normalized-losses* en el cuadro de texto.

1. En la esquina inferior derecha, seleccione **Save** (Guardar) para cerrar el selector de columnas.

    ![Excluir una columna](./media/tutorial-designer-automobile-price-train-score/exclude-column.png)

1. Seleccione el módulo **Select Columns in Dataset**. 

1. En el panel de detalles del módulo situado a la derecha del lienzo, seleccione el cuadro de texto **Comentario** y escriba *Exclude normalized losses* (Excluir normalized-losses).

    Los comentarios aparecerán en el gráfico para ayudarle a organizar la canalización.

### <a name="clean-missing-data"></a>Limpiar datos que faltan

Después de quitar la columna **normalized-losses**, aún faltan valores en el conjunto de datos. Puede quitar el resto de los datos que faltan mediante el módulo **Clean Missing Data** (Limpiar datos que faltan).

> [!TIP]
> Un requisito previo para usar la mayoría de los módulos del diseñador es limpiar los valores que faltan.

1. En la paleta de módulos que se encuentra a la izquierda del lienzo, expanda la sección **Transformación de datos** y busque el módulo **Clean Missing Data** (Limpiar datos que faltan).

1. Arrastre el módulo **Clean Missing Data** (Limpiar datos que faltan) al lienzo de la canalización. Conéctelo al módulo **Select Columns in Dataset** (Seleccionar columnas del conjunto de datos). 

1. Seleccione el módulo **Clean Missing Data** (Limpiar datos que faltan).

1. En el panel de detalles del módulo, situado a la derecha del lienzo, seleccione **Editar columna**.

1. En la ventana **Columns to be cleaned** (Columnas que se limpian) que aparece, expanda el menú desplegable situado junto a **Include** (Incluir). Seleccione **All columns** (Todas las columnas).

1. Seleccione **Guardar**.

1. En el panel de detalles del módulo situado a la derecha del lienzo, seleccione **Remove entire row** (Quitar toda la fila) en **Cleaning mode** (Modo de limpieza).

1. En el panel de detalles del módulo situado a la derecha del lienzo, seleccione el cuadro **Comentario** y escriba *Remove missing value rows* (Quitar filas de valores que faltan). 

    La canalización debe parecerse a esta:

    :::image type="content" source="./media/tutorial-designer-automobile-price-train-score/pipeline-clean.png"alt-text="Selección de columna":::

## <a name="train-a-machine-learning-model"></a>Entrenar un modelo de Machine Learning

Ahora que los módulos están listos para procesar los datos, puede configurar los módulos de entrenamiento.

Como lo que se desea es predecir un precio, que es un número, se puede usar un algoritmo de regresión. En este ejemplo, va a usar un modelo de regresión lineal.

### <a name="split-the-data"></a>División de los datos

La división de los datos es una tarea común en el aprendizaje automático. Dividirá los datos en dos conjuntos independientes. Uno de ellos entrenará el modelo, mientras que el otro probará su funcionamiento.

1. En la paleta de módulos, expanda la sección **Transformación de datos** y busque el módulo **Split Data** (Dividir datos).

1. Arrastre el módulo **Split Data** (Dividir datos) al lienzo de la canalización.

1. Conecte al puerto izquierdo del módulo **Clean Missing Data**(Limpiar datos que faltan) al módulo **Split Data** (Dividir datos).

    > [!IMPORTANT]
    > Asegúrese de que el puerto de salida izquierdo de **Clean Missing Data** (Limpiar datos que faltan) se conecta a **Split Data** (Dividir datos). El puerto izquierdo contiene los datos limpios. El puerto izquierdo contiene los datos descartados.

1. Seleccione el módulo **Split Data** (Dividir datos).

1. En el panel de detalles del módulo que se encuentra a la derecha del lienzo, establezca la opción **Fraction of rows in the first output dataset** (Fracción de filas del primer conjunto de datos de salida) en 0,7.

    De este modo divide los datos: el 70 % para entrenar el modelo y el 30 % para probarlo. Al conjunto de datos del 70 % se podrá acceder a través del puerto de salida de la izquierda. Los restantes datos estarán disponibles a través del puerto de salida derecho.

1. En el panel de detalles del módulo situado a la derecha del lienzo, seleccione el cuadro **Comentario** y escriba *Split the dataset into training set (0.7) and test set (0.3)* (Dividir el conjunto de datos en conjunto de entrenamiento [0,7] y conjunto de pruebas [0,3]).

### <a name="train-the-model"></a>Entrenamiento del modelo

Para entrenar el modelo, proporciónele un conjunto de datos que incluya el precio. El algoritmo construye un modelo que explica la relación entre las características y el precio presentado por los datos de entrenamiento.

1. En la paleta de módulos expanda **Machine Learning Algorithms** (Algoritmos de aprendizaje automático).
    
    Esta opción muestra varias categorías de módulos que se pueden usar para inicializar algoritmos de aprendizaje.

1. Seleccione **Regression** > **Linear Regression** (Regresión > Regresión lineal) y arrástrelo al lienzo de la canalización.

1. En la paleta de módulos, expanda la sección **Entrenamiento de módulos** y arrastre el módulo **Train Model** (Entrenar modelo) al lienzo.

1. Conecte la salida del módulo **Linear Regression** (Regresión lineal) a la entrada izquierda del módulo **Train Model** (Entrenar modelo).

1. Conecte la salida de datos de entrenamiento (puerto izquierdo) del módulo **Dividir datos** a la entrada derecha del módulo **Entrenar modelo**.
    
    > [!IMPORTANT]
    > Asegúrese de que el puerto de salida izquierdo de **Split Data** (Dividir datos) se conecta a **Train Model** (Entrenar modelo). El puerto izquierdo contiene el conjunto de entrenamiento. El puerto derecho contiene el conjunto de prueba.

    :::image type="content" source="./media/tutorial-designer-automobile-price-train-score/pipeline-train-model.png"alt-text="Captura de pantalla que muestra la configuración correcta del módulo Train Model (Entrenar modelo). El módulo Linear Regression (Regresión lineal) se conecta al puerto izquierdo del módulo Train Model (Entrenar modelo) y el módulo Split Data (Dividir datos) se conecta al puerto derecho de Train Model (Entrenar modelo).":::

1. Seleccione el módulo **Train Model** (Entrenar modelo).

1. En el panel de detalles del módulo, situado a la derecha del lienzo, elija el selector **Editar columna**.

1. En el cuadro de diálogo **Label column** (Columna de etiqueta), expanda el menú desplegable y seleccione **Column names** (Nombres de columna). 

1. En el cuadro de texto, escriba *precio* para especificar el valor que el modelo va a predecir.

    >[!IMPORTANT]
    > Asegúrese de escribir exactamente el nombre de la columna. No use mayúsculas para el **precio**. 

    La canalización debe ser parecida a esta:

    :::image type="content" source="./media/tutorial-designer-automobile-price-train-score/pipeline-train-graph.png"alt-text="Captura de pantalla que muestra la configuración correcta de la canalización después de agregar el módulo Train Model (Entrenar modelo).":::

### <a name="add-the-score-model-module"></a>Adición del módulo Score Model (Puntuar modelo)

Después de entrenar el modelo con el 70 % de los datos, puede usarlo para puntuar el otro 30 % y ver si el modelo funciona correctamente.

1. Escriba *score model* en el cuadro de búsqueda para encontrar el módulo **Score Model** (Puntuar modelo). Arrastre el módulo al lienzo de la canalización. 

1. Conecte la salida del módulo **Train Model** (Entrenar modelo) al puerto de entrada izquierdo de **Score Model** (Entrenar modelo). Conecte la salida de los datos de prueba (puerto derecho) del módulo **Split Data** al puerto de entrada derecho de **Score Model**.

### <a name="add-the-evaluate-model-module"></a>Agregar el módulo Evaluate Model (Evaluar modelo)

Use el módulo **Evaluate Model** (Evaluar modelo) para evaluar la puntuación que dio el modelo al conjunto de datos de prueba.

1. Escriba *evaluate* en el cuadro de búsqueda para buscar el módulo **Evaluate Model** (Evaluar modelo). Arrastre el módulo al lienzo de la canalización. 

1. Conecte la salida del módulo **Score Model** (Puntuar modelo) a la entrada izquierda de **Evaluate Model** (Evaluar modelo). 

    La canalización final debe parecerse a esta:

    :::image type="content" source="./media/tutorial-designer-automobile-price-train-score/pipeline-final-graph.png"alt-text="Captura de pantalla que muestra la configuración correcta de la canalización.":::

## <a name="submit-the-pipeline"></a>Enviar la canalización

Ahora que ya ha configurado la canalización, puede enviar una ejecución de canalización para entrenar el modelo de Machine Learning. Puede enviar una ejecución de canalización válida en cualquier momento, que se puede usar para revisar los cambios de la canalización durante el desarrollo.

1. En la parte superior del lienzo, seleccione **Enviar**.

1. En el cuadro de diálogo **Set up pipeline run** (Configurar ejecución de canalización), seleccione **Create new** (Crear nuevo).

    > [!NOTE]
    > Los experimentos agrupan ejecuciones de canalización. Si ejecuta una canalización varias veces, puede seleccionar el mismo experimento para ejecuciones sucesivas.

    1. Escriba un nombre descriptivo en **Nombre de nuevo experimento**.

    1. Seleccione **Submit** (Enviar).
    
    Puede ver el estado y los detalles de la ejecución en la parte superior derecha del lienzo.
    
    Si es la primera ejecución, la canalización puede tardar hasta 20 minutos en finalizar. La configuración del proceso predeterminada tiene un tamaño de nodo mínimo de 0, lo que significa que el diseñador debe asignar recursos después de estar inactivo. Las ejecuciones de canalización repetidas tardarán menos en terminar, dado que los recursos del proceso ya están asignados. Además, el diseñador usa resultados almacenados en la caché en cada módulo para mejorar aún más la eficiencia.

### <a name="view-scored-labels"></a>Visualización de etiquetas con puntuación

Una vez finalizada la ejecución, puede ver los resultados de la ejecución de la canalización. En primer lugar, examine las predicciones generadas por el modelo de regresión.

1. Haga clic con el botón derecho en el módulo **Score Model** (Puntuar modelo) y seleccione **Visualizar** para ver el resultado.

    Aquí puede ver los precios previstos y los precios reales de los datos de prueba.

    :::image type="content" source="./media/tutorial-designer-automobile-price-train-score/score-result.png"alt-text="Captura de pantalla de la visualización de salida en la que aparece la columna Scored Label (Etiqueta puntuada) resaltada":::

### <a name="evaluate-models"></a>Evaluación de modelos

Use **Evaluate Model** (Evaluar modelo) para ver el rendimiento del modelo entrenado en el conjunto de datos de prueba.

1. Haga clic con el botón derecho en el módulo **Evaluate Model** (Evaluar modelo) y seleccione **Visualizar** para ver el resultado.

Se muestran las siguientes estadísticas de su modelo:

* **Desviación media (MAE)** : la media de errores absolutos. Un error es la diferencia entre el valor previsto y el valor real.
* **Error cuadrático medio (RMSE)** : la raíz cuadrada de la media de errores al cuadrado de las predicciones realizadas sobre el conjunto de datos de prueba.
* **Error absoluto relativo**: la media de errores absolutos en relación con la diferencia absoluta entre los valores reales y la media de todos los valores reales.
* **Error al cuadrado relativo**: la media de errores al cuadrado en relación con la diferencia al cuadrado entre los valores reales y la media de todos los valores reales.
* **Coeficiente de determinación**: conocido también como valor R cuadrado, es una métrica estadística que indica en qué medida se ajusta un modelo a los datos.

Para cada una de las estadísticas de errores, cuanto menor sea el valor, mejor. Un valor inferior indica que las predicciones están más próximas a los valores reales. En el coeficiente de determinación, cuanto más próximo es su valor a uno (1,0), mejores son las predicciones.

## <a name="clean-up-resources"></a>Limpieza de recursos

Omita esta sección si desea continuar con la parte 2 del tutorial sobre la [implementación de modelos](tutorial-designer-automobile-price-deploy.md).

[!INCLUDE [aml-ui-cleanup](../../includes/aml-ui-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En la segunda parte, aprenderá a implementar el modelo como un punto de conexión en tiempo real.

> [!div class="nextstepaction"]
> [Continuar con la implementación de modelos](tutorial-designer-automobile-price-deploy.md)
