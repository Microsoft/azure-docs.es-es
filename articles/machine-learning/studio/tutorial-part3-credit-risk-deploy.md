---
title: 'Tutorial 3 de ML Studio (clásico): Implementación de los modelos de riesgo crediticio: Azure'
description: Tutorial detallado que muestra cómo crear una solución de análisis predictivo para la evaluación del riesgo de crédito en Azure Machine Learning Studio (clásico). Se trata de la tercera parte de un tutorial de tres. Muestra cómo se implementa un modelo como servicio web.
keywords: credit risk, predictive analytics solution,risk assessment, deploy, web service
author: likebupt
ms.author: keli19
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: tutorial
ms.date: 07/27/2020
ms.openlocfilehash: 116d939368626cadab7be8b5295b1c74f446ac27
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87431729"
---
# <a name="tutorial-3-deploy-credit-risk-model---azure-machine-learning-studio-classic"></a>Tutorial 3: Implementación de un modelo de riesgo crediticio: Azure Machine Learning Studio (clásico)

**SE APLICA A:**  ![yes](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


En este tutorial se explica con detalle el proceso de desarrollo de una solución de análisis predictivo. Va a desarrollar un modelo sencillo en Machine Learning Studio (clásico).  Después puede implementar el modelo como un servicio web de Azure Machine Learning.  Este modelo implementado puede hacer predicciones con datos nuevos. Se trata de la **tercera parte de un tutorial de tres**.

Suponga que necesita predecir el riesgo de crédito de un individuo en función de la información que se proporcionó en una solicitud de crédito.  

La evaluación de riesgos crediticios es un problema complejo, pero en este tutorial se simplificará un poco. Se utilizará como ejemplo de cómo puede crear una solución de análisis predictivo con Microsoft Azure Machine Learning Studio (clásico). En esta solución se usará Azure Machine Learning Studio (clásico) y un servicio web Machine Learning. 

En este tutorial de tres partes, vamos a comenzar con los datos de riesgo crediticio disponibles públicamente.  Después, desarrollaremos y entrenaremos un modelo predictivo.  Finalmente, vamos a implementar el modelo como servicio web.

En la [parte uno del tutorial](tutorial-part1-credit-risk.md), creó un área de trabajo de Machine Learning Studio (clásico), cargó datos y creó un experimento.

En la [segunda parte del tutorial](tutorial-part2-credit-risk-train.md) entrenó y evaluó modelos.

En esta parte del tutorial, se va a ver lo siguiente:

> [!div class="checklist"]
> * Preparar la implementación
> * Implementación del servicio web
> * Prueba del servicio web
> * Administración del servicio web
> * Acceso al servicio web

## <a name="prerequisites"></a>Requisitos previos

Completar la [parte dos del tutorial](tutorial-part2-credit-risk-train.md).

## <a name="prepare-for-deployment"></a>Preparar la implementación
Para permitir que otros usuarios puedan usar el modelo predictivo desarrollado en este tutorial, se puede implementar como servicio web en Azure.

Hasta ahora hemos estado experimentando con el entrenamiento de nuestro modelo. Sin embargo, el servicio implementado ya no va a realizar el entrenamiento: va a generar nuevas predicciones mediante la puntuación de la entrada del usuario en función de nuestro modelo. Por lo tanto, hay que realizar unos cuantos preparativos para convertir este experimento de ***entrenamiento*** en un experimento ***predictivo***. 

La preparación para la implementación es un proceso de tres pasos:  

1. Eliminación de uno de los modelos
1. Conversión del *experimento de entrenamiento* que hemos creado en un *experimento predictivo*
1. Implementar el experimento predictivo como servicio web

### <a name="remove-one-of-the-models"></a>Eliminación de uno de los modelos

En primer lugar, debemos reducir este experimento un poco. Actualmente, hay dos modelos distintos en el experimento, pero solo queremos usar uno al implementar esto como servicio web.  

Pongamos que decidimos que el modelo de árbol ampliado ofrece un rendimiento mayor que el modelo SVM. Por tanto, lo primero que se debe hacer es eliminar el módulo [Two-Class Support Vector Machine][two-class-support-vector-machine] (Máquina de vectores de soporte de dos clases) y los módulos que se usaron para entrenarlo. Puede que desee hacer una copia del experimento antes; para ello, haga clic en **Guardar como** en la parte inferior del lienzo de experimento.

Es necesario eliminar los siguientes módulos:  

* [Two-Class Support Vector Machine][two-class-support-vector-machine] (Máquina de vectores de soporte de dos clases)
* Módulos [Entrenar modelo][train-model] y [Score Model][score-model] (Puntuar modelo) conectados a él
* [Normalize Data][normalize-data] (Normalizar datos) (ambos)
* [Evaluate Model][evaluate-model] (Evaluar modelo) (porque se ha terminado de evaluar los modelos)

Seleccione cada módulo y presione la tecla Supr o haga clic con el botón derecho en el módulo y seleccione **Eliminar**. 

![Resalta los módulos que se van a eliminar para quitar el modelo de máquina de vectores de soporte](./media/tutorial-part3-credit-risk-deploy/publish3a.png)

Nuestro modelo debería tener ahora un aspecto similar al siguiente:

![Experimento resultante cuando se elimina el modelo de máquina de vectores de soporte](./media/tutorial-part3-credit-risk-deploy/publish3.png)

Ahora ya estamos preparados para implementar este modelo con el [Árbol de decisión promovido por dos clases][two-class-boosted-decision-tree].

### <a name="convert-the-training-experiment-to-a-predictive-experiment"></a>Convertir un experimento de entrenamiento en experimento predictivo

Para tener este modelo listo para la implementación, tenemos que convertir este experimento de entrenamiento en un experimento de predicción. Esto implica tres pasos:

1. Guardar el modelo que ha entrenado y sustituir nuestros módulos de aprendizaje
1. Recortar el experimento para quitar los módulos que fueron necesarios solo para el entrenamiento.
1. Definir el lugar en el que el servicio web aceptará la entrada y en el que generará la salida

Se puede hacer manualmente, pero afortunadamente, para realizar estos tres pasos, basta con hacer clic en la opción **Set Up Web Service** (Configurar servicio web) de la parte inferior del lienzo del experimento (y seleccionar la opción **Predictive web Service** [Servicio web predictivo]).

> [!TIP]
> Si desea más detalles sobre lo que ocurre cuando convierte un experimento de entrenamiento en uno de predicción, consulte el artículo sobre la [preparación del modelo de implementación en Azure Machine Learning Studio (clásico)](convert-training-experiment-to-scoring-experiment.md).

Al hacer clic en **Set Up Web Service**(Configurar servicio web), pasa lo siguiente:

* El modelo entrenado se convierte en un único módulo **Trained Model** (Modelo entrenado) y se guarda en la paleta de módulos que se encuentra a la izquierda del lienzo de experimento (se encuentra en **Modelos entrenados**).
* Se quitan los módulos que se utilizaron para el entrenamiento, en particular:
  * [Two-Class Boosted Decision Tree][two-class-boosted-decision-tree] (Árbol de decisión promovido por dos clases)
  * [Train Model][train-model] (entrenar modelo)
  * [División de datos][split]
  * El segundo módulo [Execute R Script][execute-r-script] (Ejecutar script R) que se usó para probar los datos
* El modelo entrenado guardado se vuelve a agregar al experimento.
* Se agregan los módulos **Web service input** (Entrada al servicio web) y **Web service output** (Salida de servicio web), que identifican dónde los datos del usuario especificarán el modelo y qué datos se devolverán, cuando se accede al servicio web.

> [!NOTE]
> Puede ver que el experimento se guarda en dos partes en pestañas que se han agregado en la parte superior del lienzo del experimento. El experimento de entrenamiento original está en la pestaña **Experimento de entrenamiento**, y el experimento de predicción recién creado está en **Experimento predictivo**. El experimento predictivo es el que se va a implementar como servicio web.

Necesitamos realizar un paso adicional con este experimento concreto.
Se han agregado dos módulos [Execute R Script][execute-r-script] (Ejecutar script R) para proporcionar una función de ponderación a los datos. Era simplemente un truco necesario para el entrenamiento y las pruebas, por lo que es posible quitar dichos módulos en el modelo final.
Machine Learning Studio (clásico) quitó un módulo [Execute R Script][execute-r-script] (Ejecutar script R) cuando quitó el módulo [Split][split] (Dividir). Ahora se puede quitar el otro y conectar [Metadata Editor][metadata-editor] (Editor de metadatos) directamente a [Score Model][score-model] (Puntuar modelo).    

Nuestro experimento debería tener ahora un aspecto similar al siguiente:  

![Scoring the trained model](./media/tutorial-part3-credit-risk-deploy/publish4.png)


> [!NOTE]
> Quizás se pregunte por qué hemos dejado el conjunto de datos de la tarjeta de crédito alemana de UCI en el experimento predictivo. El servicio va a utilizar los datos del usuario, no el conjunto de datos original; entonces, ¿por qué se deja el conjunto de datos original en el modelo?
> 
> Es cierto que el servicio no necesita los datos originales de la tarjeta de crédito. Pero sí necesita el esquema para esos datos, que incluye información como la cantidad de columnas que hay y cuáles son numéricas. Esta información del esquema es necesaria para interpretar los datos del usuario. Dejamos estos componentes conectados para que el módulo de puntuación tenga el esquema del conjunto de datos cuando el servicio se esté ejecutando. No se utilizan los datos, sino solamente el esquema.  
> 
>Una cosa importante a tener en cuenta es que si el conjunto de datos original contiene la etiqueta, el esquema esperado de la entrada web también espera una columna con la etiqueta. Una forma de evitarlo es quitar la etiqueta y cualquier otro dato que esté en el conjunto de datos de entrenamiento, pero que no esté en las entradas web, antes de conectar la entrada web y el conjunto de datos de entrenamiento en un módulo común. 
> 

Ejecute el experimento por última vez (haga clic en **Ejecutar**). Si desea comprobar que el modelo sigue funcionando, haga clic en la salida del módulo [Score Model][score-model] (Puntuar modelo) y seleccione **Ver resultados**. Puede ver que aparecen los datos originales, junto con el valor de riesgo de crédito (las "etiquetas puntuadas") y el valor de probabilidad de la puntuación (las "probabilidades puntuadas"). 

## <a name="deploy-the-web-service"></a>Implementación del servicio web
Puede implementar el experimento como un servicio web clásico o como un servicio web nuevo basado en Azure Resource Manager.

### <a name="deploy-as-a-classic-web-service"></a>Implementación como servicio web clásico
Para implementar un servicio web clásico derivado de nuestro experimento, haga clic en **Deploy Web Service** (Implementar servicio web) debajo del lienzo y seleccione **Deploy Web Service [Classic]** (Implementar servicio web [clásico]). Machine Learning Studio (clásico) implementa el experimento como servicio web y lo remite al panel del servicio web. Desde esta página, puede volver al experimento (**Ver instantánea** o **View latest** [Ver más reciente]) y ejecutar una prueba sencilla del servicio web (consulte la sección **Prueba del servicio web** a continuación). Aquí también hay información para crear aplicaciones que puedan acceder al servicio web (más información al respecto en el siguiente paso de este tutorial).

![Panel del servicio web](./media/tutorial-part3-credit-risk-deploy/publish6.png)


Puede configurar el servicio haciendo clic en la pestaña **CONFIGURACIÓN** . Aquí puede modificar el nombre del servicio (recibe de manera predeterminada el nombre del experimento) y proporcionarle una descripción. También puede poner etiquetas más descriptivas para los datos de entrada y salida.  

![Configurar el servicio web](./media/tutorial-part3-credit-risk-deploy/publish5.png)


### <a name="deploy-as-a-new-web-service"></a>Implementación como servicio web nuevo

> [!NOTE] 
> Para implementar un servicio web nuevo, debe tener permisos suficientes en la suscripción en la que lo implementa. Para obtener más información, consulte [Administración de un servicio web mediante el portal Servicios web Azure Machine Learning](manage-new-webservice.md). 

Para implementar un servicio web nuevo derivado del experimento:

1. Haga clic en **Deploy Web Service** (Implementar servicio web) debajo del lienzo y seleccione **Deploy Web Service [New]** (Implementar servicio web [nuevo]). Machine Learning Studio (clásico) le transfiere a la página **Deploy Experiment** (Implementar experimento) del portal de servicios web de Azure Machine Learning.

1. Escriba un nombre para el servicio web. 

1. Para **Plan de precios**, puede seleccionar un plan de precios existente, o seleccionar "Crear nuevo" y asignar un nombre al nuevo plan y seleccionar la opción de plan mensual. Los niveles de los planes predeterminados son los de la región predeterminada, y el servicio web se implementa en dicha región.

1. Haga clic en **Implementar**.

Después algunos minutos, se abre la página **Inicio rápido** del servicio web.

Puede configurar el servicio haciendo clic en la pestaña **Configurar**. Aquí puede modificar el título del servicio y escribir una descripción. 

Para probar el servicio web, haga clic en la pestaña **Probar** (vea la sección **Prueba del servicio web** a continuación). Para información sobre cómo crear aplicaciones que puedan acceder al servicio web, haga clic en la pestaña **Consumir** (en el siguiente paso de este tutorial puede encontrar más información).

> [!TIP]
> Puede actualizar el servicio web después de haberlo implementado. Por ejemplo, si desea cambiar el modelo, edite el experimento de entrenamiento, ajuste los parámetros del modelo, haga clic en **Deploy Web Service** (Implementar servicio web) y seleccione **Deploy Web Service [Classic]** (Implementar servicio web [clásico]) o **Deploy Web Service [New]** (Implementar servicio web [nuevo]). Cuando implementa el experimento de nuevo, este sustituye al servicio web, ahora con el modelo actualizado.  
> 
> 

## <a name="test-the-web-service"></a>Prueba del servicio web

Cuando se accede al servicio web, los datos del usuario se escriben con el módulo **Web service input** (Entrada al servicio web), donde se pasan al módulo [Score Model][score-model] (Puntuar modelo) y se puntúan. Según se ha configurado el experimento predictivo, el modelo espera los datos en el mismo formato que el conjunto de datos de riesgo crediticio original.
Los resultados se devuelven al usuario desde el servicio web a través del módulo **Web service output** (Salida del servicio web).

> [!TIP]
> La forma en que se configura el experimento predictivo permitirá que se devuelvan los resultados desde el módulo [Score Model][score-model] (Puntuar modelo). Esto incluye todos los datos de entrada además del valor de riesgo de crédito y la probabilidad de puntuación. Pero puede devolver algo distinto si lo desea; por ejemplo, podría devolver solamente el valor de riesgo de crédito. Para ello, inserte un módulo [Seleccionar columnas][select-columns] entre [Puntuar modelo][score-model] y **Web service output** (Salida del servicio web) para eliminar aquellas columnas que no desee que devuelva el servicio web. 
> 
> 

Puede probar el servicio web clásico en **Machine Learning Studio (clásico)** o en el portal de **Servicios web de Azure Machine Learning**.
Puede probar un servicio web nuevo en el portal **Servicios web Machine Learning**.

> [!TIP]
> Al realizar pruebas en el portal Servicios web Azure Machine Learning, puede habilitar el portal para que cree datos de ejemplo que se puedan usar para probar el servicio de solicitud-respuesta. En la página **Configurar**, seleccione "Yes" (Sí) para **Sample Data Enabled?** (¿Datos de ejemplo habilitados?). Al abrir la pestaña Solicitud-respuesta en la página **Probar**, el portal rellena los datos de ejemplo obtenidos del conjunto de datos de riesgo de crédito original.

### <a name="test-a-classic-web-service"></a>Prueba de un servicio web clásico

Puede probar el servicio web clásico en Machine Learning Studio (clásico) o en el portal de Servicios web de Azure Machine Learning. 

#### <a name="test-in-machine-learning-studio-classic"></a>Prueba de Machine Learning Studio (clásico)

1. En la página **PANEL** del servicio web, haga clic en el botón **Probar** de **Default Endpoint** (Punto de conexión predeterminado). Aparecerá un cuadro de diálogo que le pide los datos de entrada del servicio. Se trata de las mismas columnas que aparecieron en el conjunto de datos original de riesgo de crédito original.  

1. Escriba un conjunto de datos y haga clic en **Aceptar**. 

#### <a name="test-in-the-machine-learning-web-services-portal"></a>Prueba en el portal Servicios web Machine Learning

1. En la página **PANEL** del servicio web, haga clic en el botón **Test preview** (Vista preliminar de la prueba) de **Default Endpoint** (Punto de conexión predeterminado). La página de prueba del portal Servicios web Azure Machine Learning para el punto de conexión del servicio web se abrirá y le pedirá los datos de entrada para el servicio. Se trata de las mismas columnas que aparecieron en el conjunto de datos original de riesgo de crédito original.

2. Haga clic en **Test Request-Response** (Probar solicitud-respuesta). 

### <a name="test-a-new-web-service"></a>Prueba de un servicio web nuevo

Puede probar un servicio web nuevo en el portal Servicios web Machine Learning.

1. En el portal [Servicios web Azure Machine Learning](https://services.azureml.net/quickstart), haga clic en **Probar** en la parte superior de la página. Se abrirá la página **Prueba** y podrá escribir datos para el servicio. Los campos de entrada que se muestran corresponden a las columnas que aparecieron en el conjunto de datos original de riesgo de crédito. 

1. Escriba un conjunto de datos y, después, haga clic en **Test Request-Response**(Probar solicitud-respuesta).

Los resultados de la prueba se muestran en el lado derecho de la página en la columna de salida. 


## <a name="manage-the-web-service"></a>Administración del servicio web

Cuando se haya implementado el servicio web, ya sea clásico o nuevo, podrá administrarlo desde el portal [Servicios web Microsoft Azure Machine Learning](https://services.azureml.net/quickstart).

Para supervisar el rendimiento del servicio web, siga estos pasos:

1. Inicie sesión en el portal [Servicios web Microsoft Azure Machine Learning](https://services.azureml.net/quickstart).
1. Haga clic en **Servicios web**.
1. Haga clic en el servicio web.
1. Haga clic en **Panel**.

## <a name="access-the-web-service"></a>Acceso al servicio web

En el paso anterior de este tutorial hemos implementado un servicio web que usa nuestro modelo de predicción del riesgo crediticio. Ahora los usuarios pueden enviar datos al servicio y recibir resultados. 

El servicio web es un servicio web de Azure que puede recibir y devolver datos con las API de REST de una de estas dos maneras:  

* **Solicitud/respuesta** : el usuario envía una o varias filas de datos de crédito al servicio mediante un protocolo HTTP, y el servicio responde con uno o más conjuntos de resultados.
* **Ejecución de lotes** : el usuario almacena una o varias filas de datos de crédito en un blob de Azure y luego envía la ubicación del blob al servicio. El servicio puntúa todas las filas de datos en el blob de entrada, almacena los resultados en otro blog y devuelve la dirección URL del contenedor. 

> [!NOTE]
> Los nombres de las columnas de características en Studio (clásico) **distinguen mayúsculas de minúsculas**. Asegúrese de que los datos de entrada para invocar el servicio web tengan los mismos nombres de columna que en el conjunto de datos de entrenamiento.

Para más información acerca el acceso y consumo del servicio web, consulte [Consumo de un servicio web de Azure Machine Learning con una plantilla de aplicación web](/azure/machine-learning/studio/consume-web-services).



## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [machine-learning-studio-clean-up](../../../includes/machine-learning-studio-clean-up.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha completado estos pasos:

> [!div class="checklist"]
> * Preparar la implementación
> * Implementación del servicio web
> * Prueba del servicio web
> * Administración del servicio web
> * Acceso al servicio web

También puede desarrollar una aplicación personalizada para acceder al servicio web con código de inicio proporcionado en los lenguajes de programación R, C# y Python.

> [!div class="nextstepaction"]
> [Consumo de un servicio web Azure Machine Learning](consume-web-services.md)

<!-- Module References -->
[evaluate-model]: https://msdn.microsoft.com/library/azure/927d65ac-3b50-4694-9903-20f6c1672089/
[execute-r-script]: https://msdn.microsoft.com/library/azure/30806023-392b-42e0-94d6-6b775a6e0fd5/
[metadata-editor]: https://msdn.microsoft.com/library/azure/370b6676-c11c-486f-bf73-35349f842a66/
[normalize-data]: https://msdn.microsoft.com/library/azure/986df333-6748-4b85-923d-871df70d6aaf/
[score-model]: https://msdn.microsoft.com/library/azure/401b4f92-e724-4d5a-be81-d5b0ff9bdb33/
[split]: https://msdn.microsoft.com/library/azure/70530644-c97a-4ab6-85f7-88bf30a8be5f/
[train-model]: https://msdn.microsoft.com/library/azure/5cc7053e-aa30-450d-96c0-dae4be720977/
[two-class-boosted-decision-tree]: https://msdn.microsoft.com/library/azure/e3c522f8-53d9-4829-8ea4-5c6a6b75330c/
[two-class-support-vector-machine]: https://msdn.microsoft.com/library/azure/12d8479b-74b4-4e67-b8de-d32867380e20/
[select-columns]: https://msdn.microsoft.com/library/azure/1ec722fa-b623-4e26-a44e-a50c6d726223/
