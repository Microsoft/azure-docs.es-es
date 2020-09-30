---
title: Evaluación de la equidad de los modelos de ML en Python (versión preliminar)
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo evaluar la equidad de los modelos en Azure Machine Learning
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: mesameki
author: mesameki
ms.reviewer: luquinta
ms.date: 09/01/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 0332400ce6808e92ba09e3bee3277495f6b6d1a9
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90897486"
---
# <a name="use-azure-machine-learning-with-the-fairlearn-open-source-package-to-assess-the-fairness-of-ml-models-preview"></a>Uso de Azure Machine Learning con el paquete de código abierto Fairlearn para evaluar la equidad de los modelos de ML (versión preliminar)



En esta guía paso a paso, aprenderá a usar el paquete Python de código abierto [Fairlearn](https://fairlearn.github.io/) con Azure Machine Learning para realizar las siguientes tareas:

* Evalúe la equidad de las predicciones del modelo. Para más información sobre la equidad en el aprendizaje automático, consulte la [equidad en el artículo del aprendizaje automático](concept-fairness-ml.md).
* Cargue, enumere y descargue información de evaluación de equidad en Azure Machine Learning Studio.
* Consulte un panel de evaluación de equidad en Azure Machine Learning Studio para interactuar con la información de equidad de los modelos.

>[!NOTE]
> La evaluación de equidad no es un ejercicio puramente técnico. **Este paquete puede ayudarle a evaluar la equidad de un modelo de aprendizaje automático, pero solo puede configurar y tomar decisiones sobre cómo funciona el modelo.**  Aunque este paquete ayuda a identificar métricas cuantitativas para evaluar la equidad, los desarrolladores de modelos de aprendizaje automático también deben realizar un análisis cualitativo para evaluar la equidad de sus propios modelos.

## <a name="azure-machine-learning-fairness-sdk"></a>SDK de equidad Azure Machine Learning 

El SDK de equidad Azure Machine Learning, `azureml-contrib-fairness`, integra el paquete de código abierto de Python, [Fairlearn](http://fairlearn.github.io), dentro de Azure Machine Learning. Para más información acerca de la integración de Fairlearn en Azure Machine Learning, consulte estos [cuadernos de muestra](https://github.com/Azure/MachineLearningNotebooks/tree/master/contrib/fairness). Para más información sobre Fairlearn, consulte la [guía de ejemplo](https://fairlearn.github.io/auto_examples/) y los [cuadernos de muestra](https://github.com/fairlearn/fairlearn/tree/master/notebooks). 

Use los siguientes comandos para instalar los paquetes `azureml-contrib-fairness` y `fairlearn`:
```bash
pip install azureml-contrib-fairness
pip install fairlearn==0.4.6
```



## <a name="upload-fairness-insights-for-a-single-model"></a>Cargar información de equidad para un único modelo

En el siguiente ejemplo se muestra cómo usar el paquete de equidad para cargar información de equidad del modelo en Azure Machine Learning y ver el panel de evaluación de equidad en Azure Machine Learning Studio.

1. Entrenar un modelo de muestra en un cuaderno de Jupyter. 

    Para el conjunto de datos, usamos el conocido conjunto de datos del censo de adultos, que cargamos con `shap` (por conveniencia). Para los fines de este ejemplo, trataremos este conjunto de datos como un problema de toma de decisiones de préstamos y fingimos que la etiqueta indica si cada uno de ellos reembolsa un préstamo en el pasado. Usaremos los datos para entrenar un predicción a fin de predecir si los individuos que anteriormente no se han visto devolverán un préstamo o no. Se supone que las predicciones del modelo se utilizan para decidir si se debe ofrecer un préstamo a una persona.

    ```python
    from sklearn.model_selection import train_test_split
    from fairlearn.widget import FairlearnDashboard
    from sklearn.linear_model import LogisticRegression
    from sklearn.preprocessing import LabelEncoder, StandardScaler
    import pandas as pd
    import shap

    # Load the census dataset
    X_raw, Y = shap.datasets.adult()
    X_raw["Race"].value_counts().to_dict()
    

    # (Optional) Separate the "sex" and "race" sensitive features out and drop them from the main data prior to training your model
    A = X_raw[['Sex','Race']]
    X = X_raw.drop(labels=['Sex', 'Race'],axis = 1)
    X = pd.get_dummies(X)
    
    sc = StandardScaler()
    X_scaled = sc.fit_transform(X)
    X_scaled = pd.DataFrame(X_scaled, columns=X.columns)

    # Perform some standard data preprocessing steps to convert the data into a format suitable for the ML algorithms
    le = LabelEncoder()
    Y = le.fit_transform(Y)

    # Split data into train and test
    from sklearn.model_selection import train_test_split
    from sklearn.model_selection import train_test_split
    X_train, X_test, Y_train, Y_test, A_train, A_test = train_test_split(X_scaled, 
                                                        Y, 
                                                        A,
                                                        test_size = 0.2,
                                                        random_state=0,
                                                        stratify=Y)

    # Work around indexing issue
    X_train = X_train.reset_index(drop=True)
    A_train = A_train.reset_index(drop=True)
    X_test = X_test.reset_index(drop=True)
    A_test = A_test.reset_index(drop=True)

    # Improve labels
    A_test.Sex.loc[(A_test['Sex'] == 0)] = 'female'
    A_test.Sex.loc[(A_test['Sex'] == 1)] = 'male'


    A_test.Race.loc[(A_test['Race'] == 0)] = 'Amer-Indian-Eskimo'
    A_test.Race.loc[(A_test['Race'] == 1)] = 'Asian-Pac-Islander'
    A_test.Race.loc[(A_test['Race'] == 2)] = 'Black'
    A_test.Race.loc[(A_test['Race'] == 3)] = 'Other'
    A_test.Race.loc[(A_test['Race'] == 4)] = 'White'


    # Train a classification model
    lr_predictor = LogisticRegression(solver='liblinear', fit_intercept=True)
    lr_predictor.fit(X_train, Y_train)

    # (Optional) View this model in Fairlearn's fairness dashboard, and see the disparities which appear:
    from fairlearn.widget import FairlearnDashboard
    FairlearnDashboard(sensitive_features=A_test, 
                       sensitive_feature_names=['Sex', 'Race'],
                       y_true=Y_test,
                       y_pred={"lr_model": lr_predictor.predict(X_test)})
    ```

2. Inicie sesión en Azure Machine Learning y registre el modelo.
   
    El panel de equidad puede integrarse con modelos registrados o no registrados. Registre el modelo en Azure Machine Learning con los siguientes pasos:
    ```python
    from azureml.core import Workspace, Experiment, Model
    import joblib
    import os
    
    ws = Workspace.from_config()
    ws.get_details()

    os.makedirs('models', exist_ok=True)
    
    # Function to register models into Azure Machine Learning
    def register_model(name, model):
        print("Registering ", name)
        model_path = "models/{0}.pkl".format(name)
        joblib.dump(value=model, filename=model_path)
        registered_model = Model.register(model_path=model_path,
                                        model_name=name,
                                        workspace=ws)
        print("Registered ", registered_model.id)
        return registered_model.id

    # Call the register_model function 
    lr_reg_id = register_model("fairness_logistic_regression", lr_predictor)
    ```

3. Calcular previamente las métricas de equidad.

    Cree un diccionario de panel con el paquete `metrics` de Fairlearn. El método `_create_group_metric_set` tiene argumentos similares al constructor del panel, salvo que las características confidenciales se pasan como un diccionario (para asegurarse de que los nombres están disponibles). También se debe especificar el tipo de predicción (clasificación binaria en este caso) cuando se llama a este método.

    ```python
    #  Create a dictionary of model(s) you want to assess for fairness 
    sf = { 'Race': A_test.Race, 'Sex': A_test.Sex}
    ys_pred = { lr_reg_id:lr_predictor.predict(X_test) }
    from fairlearn.metrics._group_metric_set import _create_group_metric_set

    dash_dict = _create_group_metric_set(y_true=Y_test,
                                        predictions=ys_pred,
                                        sensitive_features=sf,
                                        prediction_type='binary_classification')
    ```
4. Cargue las métricas de equidad precalculadas.
    
    Ahora, importe el paquete `azureml.contrib.fairness` para realizar la carga:

    ```python
    from azureml.contrib.fairness import upload_dashboard_dictionary, download_dashboard_by_upload_id
    ```
    Cree un Experimento, después una Ejecución y cargue el panel en él:
    ```python
    exp = Experiment(ws, "Test_Fairness_Census_Demo")
    print(exp)

    run = exp.start_logging()

    # Upload the dashboard to Azure Machine Learning
    try:
        dashboard_title = "Fairness insights of Logistic Regression Classifier"
        # Set validate_model_ids parameter of upload_dashboard_dictionary to False if you have not registered your model(s)
        upload_id = upload_dashboard_dictionary(run,
                                                dash_dict,
                                                dashboard_name=dashboard_title)
        print("\nUploaded to id: {0}\n".format(upload_id))
        
        # To test the dashboard, you can download it back and ensure it contains the right information
        downloaded_dict = download_dashboard_by_upload_id(run, upload_id)
    finally:
        run.complete()
    ```
5. Comprobación del panel de equidad desde Azure Machine Learning Studio

    Si completa los pasos anteriores (carga de información de equidad generada en Azure Machine Learning), puede ver el panel de equidad en [Azure Machine Learning Studio](https://ml.azure.com). Este panel es el mismo panel de visualización proporcionado en Fairlearn, lo que le permite analizar las diferencias entre los subgrupos de la característica confidencial (por ejemplo, macho frente a hembra).
    Siga una de estas rutas de acceso para llegar al panel de visualización de Azure Machine Learning Studio:

    * **Panel de experimentos (versión preliminar)**
    1. Al hacer clic en **Experimentos** en el panel izquierdo, verá una lista de experimentos que se han ejecutado en Azure Machine Learning.
    1. Seleccione un experimento determinado para ver todas las ejecuciones de ese experimento.
    1. Seleccione una ejecución y luego, la pestaña **Equidad** en el panel de visualización de la explicación.


    [![Panel de equidad](./media/how-to-machine-learning-fairness-aml/dashboard.png)](./media/how-to-machine-learning-fairness-aml/dashboard.png#lightbox)
    
    * **Panel de modelos**
    1. Si registró el modelo original siguiendo los pasos anteriores, puede seleccionar **Modelos** en el panel izquierdo para verlo.
    1. Seleccione un modelo y luego, la pestaña **Equidad** para ver el panel de visualización de la explicación.

    Para más información sobre el panel de visualización y lo que contiene, consulte la [guía del usuario](https://fairlearn.github.io/user_guide/assessment.html#fairlearn-dashboard) de Fairlearn.

## <a name="upload-fairness-insights-for-multiple-models"></a>Cargar información de equidad para varios modelos

Si está interesado en comparar varios modelos y ver cómo difieren sus evaluaciones de equidad, puede pasar más de un modelo al panel de visualización y navegar por sus ventajas e inconvenientes en el rendimiento.

1. Entrenar los modelos:
    
    Además del modelo de regresión logística anterior, ahora vamos a crear un segundo clasificador, basado en un estimador de máquina de vectores de soporte y cargamos un diccionario de panel de equidad con el paquete `metrics` de Fairlearn. Tenga en cuenta que aquí se omiten los pasos para cargar y preprocesar datos y pasar directamente a la fase de entrenamiento del modelo.


    ```python
    # Train your first classification model
    from sklearn.linear_model import LogisticRegression
    lr_predictor = LogisticRegression(solver='liblinear', fit_intercept=True)
    lr_predictor.fit(X_train, Y_train)

    # Train your second classification model
    from sklearn import svm
    svm_predictor = svm.SVC()
    svm_predictor.fit(X_train, Y_train)
    ```

2. Registrar los modelos

    Después, registre ambos modelos dentro de Azure Machine Learning. Para mayor comodidad en las llamadas a métodos posteriores, almacene los resultados en un diccionario, que asigna el `id` del modelo registrado (una cadena en formato `name:version`) al predictor mismo:

    ```python
    model_dict = {}

    lr_reg_id = register_model("fairness_logistic_regression", lr_predictor)
    model_dict[lr_reg_id] = lr_predictor

    svm_reg_id = register_model("fairness_svm", svm_predictor)
    model_dict[svm_reg_id] = svm_predictor
    ```

3. Cargar el panel de Fairlearn localmente

    Antes de cargar la información de equidad en Azure Machine Learning, puede examinar estas predicciones en un panel de Fairlearn invocado localmente. 



    ```python
    #  Generate models' predictions and load the fairness dashboard locally 
    ys_pred = {}
    for n, p in model_dict.items():
        ys_pred[n] = p.predict(X_test)

    from fairlearn.widget import FairlearnDashboard

    FairlearnDashboard(sensitive_features=A_test, 
                    sensitive_feature_names=['Sex', 'Race'],
                    y_true=Y_test.tolist(),
                    y_pred=ys_pred)
    ```

3. Calcular previamente las métricas de equidad.

    Cree un diccionario de panel con el paquete `metrics` de Fairlearn.

    ```python
    sf = { 'Race': A_test.Race, 'Sex': A_test.Sex }

    from fairlearn.metrics._group_metric_set import _create_group_metric_set

    dash_dict = _create_group_metric_set(y_true=Y_test,
                                        predictions=ys_pred,
                                        sensitive_features=sf,
                                        prediction_type='binary_classification')
    ```
4. Cargue las métricas de equidad precalculadas.
    
    Ahora, importe el paquete `azureml.contrib.fairness` para realizar la carga:

    ```python
    from azureml.contrib.fairness import upload_dashboard_dictionary, download_dashboard_by_upload_id
    ```
    Cree un Experimento, después una Ejecución y cargue el panel en él:
    ```python
    exp = Experiment(ws, "Compare_Two_Models_Fairness_Census_Demo")
    print(exp)

    run = exp.start_logging()

    # Upload the dashboard to Azure Machine Learning
    try:
        dashboard_title = "Fairness Assessment of Logistic Regression and SVM Classifiers"
        # Set validate_model_ids parameter of upload_dashboard_dictionary to False if you have not registered your model(s)
        upload_id = upload_dashboard_dictionary(run,
                                                dash_dict,
                                                dashboard_name=dashboard_title)
        print("\nUploaded to id: {0}\n".format(upload_id))
        
        # To test the dashboard, you can download it back and ensure it contains the right information
        downloaded_dict = download_dashboard_by_upload_id(run, upload_id)
    finally:
        run.complete()
    ```


    Al igual que en la sección anterior, puede seguir una de las rutas de acceso descritas anteriormente (a través de **Experimentos** o **Modelos**) en Azure Machine Learning Studio para acceder al panel de visualización y comparar los dos modelos en cuanto a equidad y rendimiento.


## <a name="upload-unmitigated-and-mitigated-fairness-insights"></a>Cargar información de equidad no mitigada y mitigada

Puede usar los [algoritmos de mitigación](https://fairlearn.github.io/user_guide/mitigation.html) de Fairlearn, comparar sus modelos mitigados generados con el modelo no mitigado original y navegar por los equilibrios de rendimiento/equidad entre los modelos comparados.

Para ver un ejemplo que demuestre el uso del algoritmo de mitigación de la [Búsqueda de cuadrícula](https://fairlearn.github.io/user_guide/mitigation.html#grid-search) (que crea una colección de modelos mitigados con diferentes equilibrios de equidad y rendimiento), consulte este [cuaderno de muestra](https://github.com/Azure/MachineLearningNotebooks/blob/master/contrib/fairness/fairlearn-azureml-mitigation.ipynb). 

La carga de la información de equidad de varios modelos en una sola ejecución permitiría la comparación de modelos con respecto a la equidad y el rendimiento. Puede hacer clic en cualquiera de los modelos mostrados en el gráfico de comparación de modelos para ver la información detallada sobre la equidad del modelo determinado.


[![Panel de comparación de modelos de Fairlearn](./media/how-to-machine-learning-fairness-aml/multi-model-dashboard.png)](./media/how-to-machine-learning-fairness-aml/multi-model-dashboard.png#lightbox)
    

## <a name="next-steps"></a>Pasos siguientes

[Más información sobre la imparcialidad de modelos](concept-fairness-ml.md)

[Consulte los cuadernos de muestra de equidad de Azure Machine Learning](https://github.com/Azure/MachineLearningNotebooks/tree/master/contrib/fairness)
