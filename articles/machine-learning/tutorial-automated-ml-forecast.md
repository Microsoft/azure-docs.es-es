---
title: 'Tutorial: Previsión de la demanda y aprendizaje automático automatizado'
titleSuffix: Azure Machine Learning
description: Aprenda a entrenar e implementar un modelo de previsión de la demanda con aprendizaje automático automatizado en Azure Machine Learning Studio.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.author: sacartac
ms.reviewer: nibaccam
author: cartacioS
ms.date: 07/10/2020
ms.openlocfilehash: 6ad3e0f3077e6f65642496d4da097fa713bddd53
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90979082"
---
# <a name="tutorial-forecast-demand-with-automated-machine-learning"></a>Tutorial: Previsión de la demanda con aprendizaje automático automatizado


En este tutorial utilizará el aprendizaje automático automatizado, o ML automatizado, en Azure Machine Learning Studio para crear un modelo de previsión de la demanda de serie temporal de la demanda de alquiler para un servicio de uso compartido de bicicletas.

Para obtener un ejemplo de modelo de clasificación, consulte [Tutorial: Creación de un modelo de clasificación con aprendizaje automático automatizado en Azure Machine Learning](tutorial-first-experiment-automated-ml.md).

En este tutorial, aprenderá las siguientes tareas:

> [!div class="checklist"]
> * Creación y carga de un conjunto de datos.
> * Configuración y ejecución de un experimento de ML automatizado.
> * Especifique la configuración de la previsión.
> * Exploración de los resultados del experimento.
> * Implementación del mejor modelo.

## <a name="prerequisites"></a>Requisitos previos

* Un área de trabajo de Azure Machine Learning. Consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md). 

* Descargue el archivo de datos [bike-no.csv](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-bike-share/bike-no.csv).

## <a name="get-started-in-azure-machine-learning-studio"></a>Introducción a Azure Machine Learning Studio

En este tutorial, se crea el experimento de aprendizaje automático automatizado que se ejecuta en Azure Machine Learning Studio, una interfaz web consolidada que incluye herramientas de aprendizaje automático para llevar a la práctica escenarios de ciencia de datos para los profesionales de ciencia de datos de todos los niveles de conocimiento. Studio no se admite en los exploradores Internet Explorer.

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com).

1. Seleccione la suscripción y el área de trabajo que ha creado.

1. Seleccione **Comenzar**.

1. Seleccione **ML automatizado** en la sección **Autor** en el panel de la izquierda.

1. Seleccione **+New automated ML run** (+Nueva ejecución de ML automatizado). 

## <a name="create-and-load-dataset"></a>Creación y carga de un conjunto de datos

Antes de configurar el experimento, cargue el archivo de datos en el área de trabajo en forma de conjunto de datos de Azure Machine Learning. Así podrá asegurarse de que los datos tienen el formato adecuado para el experimento.

1. En el formulario **Select dataset** (Seleccionar un conjunto de datos), seleccione **From local files** (De archivos locales) del menú desplegable **+Create dataset** (Crear conjunto de datos). 

    1. En el formulario **Basic info** (Información básica), asígnele un nombre al conjunto de datos y, si lo desea, incluya una descripción. De forma predeterminada, el tipo de datos es **Tabular** (Tabular), ya que el aprendizaje automático automatizado en Azure Machine Learning Studio por ahora solo admite los conjuntos de datos tabulares.
    
    1. Seleccione **Next** (Siguiente) en la parte inferior izquierda.

    1. En el formulario **Datastore and file selection** (Selección del archivo y el almacén de datos), seleccione el almacén de archivos predeterminado que se configuró automáticamente durante la creación del área de trabajo, **workspaceblobstore (Azure Blob Storage)** . Esta es la ubicación de almacenamiento donde se cargará el archivo de datos. 

    1. Haga clic en **Examinar**. 
    
    1. Seleccione el archivo **bike-no.csv** en el equipo local. Este es el archivo que descargó como [requisito previo](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-bike-share/bike-no.csv).

    1. Seleccione **Siguiente**.

       Una vez completada la carga, el formulario de configuración y vista previa se rellena de forma inteligente en función del tipo de archivo. 
       
    1. Compruebe que el formulario **Settings y Preview** (Configuración y vista previa) se rellenan como se indica a continuación y seleccione **Next** (Siguiente).
        
        Campo|Descripción| Valor para el tutorial
        ---|---|---
        Formato de archivo|Define el diseño y el tipo de datos almacenados en un archivo.| Delimitado
        Delimitador|Uno o más caracteres para especificar el límite entre&nbsp; regiones independientes en texto sin formato u otros flujos de datos. |Coma
        Encoding|Identifica qué tabla de esquema de bit a carácter se va a usar para leer el conjunto de elementos.| UTF-8
        Encabezados de columna| Indica cómo se tratarán los encabezados del conjunto de datos, si existen.| Usar encabezados del primer archivo
        Omitir filas | Indica el número de filas, si hay alguna, que se omiten en el conjunto de datos.| None

    1. El formulario **Scheme** (Esquema) permite una configuración adicional de los datos para este experimento. 
    
        1. En este ejemplo, elija omitir las columnas **casual** (ocasional) y **registered** (registrado). Estas columnas son un desglose de la columna **cnt** (recuento) por lo que no las incluimos.

        1. Además, en este ejemplo, deje los valores predeterminados para **Properties** (Propiedades) y **Type** (Tipo). 
        
        1. Seleccione **Next** (Siguiente).

    1. En el formulario **Confirm details** (Confirmar detalles) compruebe que la información coincide con lo rellenado anteriormente en los formularios **Basic info** (Información básica) y **Settings and preview** (Configuración y vista previa).

    1. Seleccione **Create** (Crear) para completar la creación del conjunto de datos.

    1. Seleccione el conjunto de datos cuando aparezca en la lista.

    1. Seleccione **Siguiente**.

## <a name="configure-experiment-run"></a>Configuración de la ejecución de un experimento

Una vez cargados y configurados los datos, configure el destino de proceso remoto y seleccione la columna de los datos que desea predecir.

1. Rellene el formulario **Configure run** (Configurar ejecución) como se indica a continuación:
    1. Escriba el nombre del experimento: `automl-bikeshare`.

    1. Seleccione **cnt** (recuento) como columna de destino en la que desea realizar las predicciones. Esta columna indica el total de alquileres de bicicletas.

    1. Seleccione **Create a new compute** (Crear un proceso) y configure el destino de proceso. ML automatizado solo admite el proceso con Azure Machine Learning. 

        Campo | Descripción | Valor para el tutorial
        ----|---|---
        Nombre del proceso |Un nombre único que identifique el contexto del proceso.|bike-compute
        Tipo de&nbsp;máquina&nbsp;virtual|Seleccione el tipo de máquina virtual del proceso.|CPU (Unidad central de procesamiento)
        Tamaño de la&nbsp;máquina&nbsp;virtual| Seleccione el tamaño de la máquina virtual para el proceso.|Standard_DS12_V2
        Nodos mín./máx.| Para generar perfiles de datos, debe especificar uno o más nodos.|Número mínimo de nodos: 1<br>Número máximo de nodos: 6
        Segundos de inactividad antes de la reducción vertical | Tiempo de inactividad antes de que el clúster se escale automáticamente hasta el número mínimo de nodos.|120 (valor predeterminado)
        Configuración avanzada | Valores para configurar y autorizar una red virtual para el experimento.| None
  
        1. Seleccione **Create** (Crear) para obtener el destino de proceso. 

            **Tarda unos minutos en completarse.** 

        1. Después de la creación, seleccione el nuevo destino de proceso en la lista desplegable.

    1. Seleccione **Next** (Siguiente).

## <a name="select-forecast-settings"></a>Seleccionar la configuración de la previsión

Complete la configuración del experimento de ML automatizado especificando el tipo de tarea de aprendizaje automático y los valores de configuración.

1. En el formulario **Task type and settings** (Configuración y tipo de tarea), seleccione **Time series forecasting** (Previsión se series temporales) como tipo de tarea de aprendizaje automático.

1. Seleccione **date** (fecha) como **Time column** (Columna de hora) y deje **Time series identifiers** (Identificadores de serie temporal) en blanco. 

1. El **horizonte de previsión** es la longitud de tiempo en el futuro que se quiere predecir.  Anule la selección de detección automática y escriba 14 en el campo. 

1. Seleccione **View additional configuration settings** (Ver opciones de configuración adicionales) y rellene los campos como se indica a continuación. Esta configuración es para controlar mejor el trabajo de entrenamiento y especificar la configuración de la previsión. De lo contrario, los valores predeterminados se aplican en función de la selección y los datos del experimento.

    Configuraciones&nbsp;adicionales|Descripción|Valor&nbsp;para&nbsp;tutorial
    ------|---------|---
    Métrica principal| Métrica de evaluación por la que se medirá el algoritmo de aprendizaje automático.|Error cuadrático medio normalizado
    Explicación del mejor modelo| Muestra automáticamente la posible explicación relativa al mejor modelo creado mediante ML automatizado.| Habilitar
    Algoritmos bloqueados | Algoritmos que desea excluir del trabajo de entrenamiento.| Árboles aleatorios extremos
    Configuración adicional de la previsión| Esta configuración ayuda a mejorar la precisión del modelo. <br><br> _**Forecast target lags**_ (Retrasos de objetivo de previsión): cuánto quiere retroceder en el tiempo para construir los retrasos de la variable de destino. <br> _**Periodos acumulados de destino**_: especifica la duración de los periodos acumulados en la que se generarán características como *max, min* (máx., mín.) y *sum* (suma). | <br><br>Retardos de&nbsp;destino&nbsp;para la previsión: None <br> Periodos&nbsp;acumulados&nbsp;de destino&nbsp;: None
    Criterios de exclusión| Si se cumplen los criterios, se detiene el trabajo de entrenamiento. |Tiempo del&nbsp;trabajo de&nbsp;entrenamiento (en horas): 3 <br> Umbral de&nbsp;puntuación&nbsp;de métrica: None
    Validación | Elija un tipo de validación cruzada y un número de pruebas.|Tipo de validación:<br>validación cruzada de&nbsp;k iteraciones&nbsp; <br> <br> Número de validaciones: 5
    Simultaneidad| Número máximo de iteraciones paralelas ejecutadas por iteración| Máximo de iteraciones&nbsp;simultáneas&nbsp;: 6
    
    Seleccione **Guardar**.

## <a name="run-experiment"></a>Ejecutar experimento

Para ejecutar el experimento, seleccione **Finish** (Finalizar). La pantalla **Run details** (Detalles de ejecución) se abrirá con **Run status** (Estado de la ejecución) en la parte superior junto al número de ejecución. Este estado se actualiza a medida que el experimento progresa.

>[!IMPORTANT]
> La preparación necesita de **10 a 15 minutos** en preparar la ejecución del experimento.
> Una vez que se ejecuta, se tarda de **2 a 3 minutos más para cada iteración**.  <br> <br>
> En producción, probablemente puede descansar un poco, ya que el proceso tarda. Mientras espera, se recomienda empezar por explorar los algoritmos probados de la pestaña **Models** (Modelos) que se van completando. 

##  <a name="explore-models"></a>Exploración de modelos

Vaya a la pestaña **Models** (Modelos) para ver los algoritmos (modelos) probados. De forma predeterminada, los modelos se ordenan por puntuación de las métricas a medida que se completan. De manera predeterminada, el modelo con la mayor puntuación según la métrica **Error cuadrático medio normalizado** elegida aparece en la parte superior de la lista.

Mientras espera a que terminen todos los modelos del experimento, seleccione **Algorithm name** (Nombre de algoritmo) de un modelo completado para explorar los detalles de rendimiento. 

El siguiente ejemplo le lleva por las pestañas **Details** (Detalles) y **Metrics** (Métricas) para ver las propiedades, las métricas y los gráficos de rendimiento del modelo seleccionado. 

![Detalles de la ejecución](./media/tutorial-automated-ml-forecast/explore-models-ui.gif)

## <a name="deploy-the-model"></a>Implementación del modelo

El aprendizaje automático automatizado en Azure Machine Learning Studio permite implementar el mejor modelo como servicio web en pocos pasos. La implementación es la integración del modelo para que pueda predecir datos nuevos e identificar posibles áreas de oportunidad. 

Para este experimento, la implementación en un servicio web significa que la empresa de uso compartido de bicicletas ahora tiene una solución web iterativa y escalable para prever la demanda de alquiler de la cuota de bicicletas. 

Una vez finalizada la ejecución, vuelva a la página de ejecución primaria seleccionando **Run 1** (Ejecución 1) en la parte superior de la pantalla.

En la sección **Best model summary** (Mejor resumen del modelo), **StackEnsemble** se considera el mejor modelo en el contexto de este experimento, según la métrica **Normalized root mean squared error** (Error de desviación media cuadrática normalizada).  

Se implementa este modelo, pero se recomienda que la implementación tarda unos 20 minutos en completarse. El proceso de implementación conlleva varios pasos, como el registro del modelo, la generación de recursos y su configuración para el servicio web.

1. Seleccione **StackEnsemble** para abrir la página específica del modelo.

1. Seleccione el botón **Deploy** (Implementar) situado en el área superior izquierda de la pantalla.

1. Rellene el panel **Deploy Model** (Implementar modelo) como se indica a continuación:

    Campo| Value
    ----|----
    Nombre de implementación| bikeshare-deploy
    Descripción de implementación| implementación de la demanda de uso compartido de bicicletas
    Compute type (Tipo de proceso) | Seleccione Azure Compute Instance (ACI) (Instancia de proceso de Azure [ACI])
    Enable authentication (Habilitar autenticación)| Deshabilitar. 
    Use custom deployment assets (Usar recursos de implementación personalizados)| Deshabilitar. La deshabilitación permite que se generen automáticamente el archivo de controlador predeterminado (script de puntuación) y el archivo de entorno. 
    
    En este ejemplo, usamos los valores predeterminados que se proporcionan en el menú *Advanced* (Avanzada). 

1. Seleccione **Implementar**.  

    Aparece un mensaje en verde en la parte superior de la pantalla **Run** (Ejecutar) que indica que la implementación se inició correctamente. El progreso de la implementación se puede encontrar en el panel **Model summary** (Resumen de modelo), en **Deploy status** (Estado de implementación).
    
Una vez finalizada la implementación correctamente, tendrá un servicio web operativo para generar predicciones. 

Continúe con [**Pasos siguientes**](#next-steps) para más información sobre el consumo del nuevo servicio web y para probar las predicciones con la compatibilidad con Azure Machine Learning incorporada en Power BI.

## <a name="clean-up-resources"></a>Limpieza de recursos

Los archivos de implementación son mayores que los archivos de datos y del experimento, por lo que cuesta más almacenarlos. Elimine solo los archivos de implementación para minimizar costos en su cuenta y si desea conservar el área de trabajo y los archivos del experimento. Elimine todo el grupo de recursos completo si no planea usar ninguno de los archivos.  

### <a name="delete-the-deployment-instance"></a>Eliminación de la instancia de implementación

Elimine solo la instancia de implementación de Azure Machine Learning Studio, si desea mantener el área de trabajo y el grupo de recursos para otros tutoriales y para explorarlos. 

1. Vaya a [Azure Machine Learning Studio](https://ml.azure.com/). Vaya al área de trabajo y, a la izquierda, en el panel **Assets** (Recursos), seleccione **Endpoints** (Puntos de conexión). 

1. Seleccione la implementación que desea eliminar y seleccione **Eliminar.** 

1. Seleccione **Continuar**.

### <a name="delete-the-resource-group"></a>Eliminar el grupo de recursos

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial se ha usado ML automatizado en Azure Machine Learning Studio para crear e implementar un modelo de previsión de series temporales que prediga la demanda de alquiler de bicicletas. 

Consulte este artículo para conocer los pasos de creación de un esquema que admita Power BI con el fin de facilitar el consumo del servicio web recién implementado:

> [!div class="nextstepaction"]
> [Consumo de un servicio web](how-to-consume-web-service.md#consume-the-service-from-power-bi)

+ Más información acerca del [aprendizaje automático automatizado](concept-automated-ml.md).
+ Para más información sobre las métricas de clasificación y los gráficos, consulte el artículo de [descripción de los resultados de aprendizaje automático automatizado](how-to-understand-automated-ml.md#classification).
+ Más información sobre la [caracterización](how-to-configure-auto-features.md#featurization).
+ Más información acerca de la [generación de perfiles de datos](how-to-connect-data-ui.md#profile).

>[!NOTE]
> Este conjunto de datos del uso compartido de bicicletas se ha modificado para el tutorial. Este conjunto de datos se puso a disponibilidad como parte de un [concurso de Kaggle](https://www.kaggle.com/c/bike-sharing-demand/data) y estaba accesible originalmente desde [Capital Bikeshare](https://www.capitalbikeshare.com/system-data). También se puede encontrar en la [base de datos UCI de Machine Learning](http://archive.ics.uci.edu/ml/datasets/Bike+Sharing+Dataset).<br><br>
> Origen: Fanaee-T, Hadi, and Gama, Joao, Event labeling combining ensemble detectors and background knowledge, Progress in Artificial Intelligence (2013): págs. 1-15, Springer Berlin Heidelberg.
