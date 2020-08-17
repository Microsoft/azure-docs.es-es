---
title: Supervisión y recopilación de datos de los puntos de conexión del servicio web Machine Learning
titleSuffix: Azure Machine Learning
description: Supervisión de los servicios web implementados con Azure Machine Learning mediante Azure App Insights
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: jmartens
ms.author: larryfr
author: blackmist
ms.date: 07/23/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: ae66447e128b07ce942b8c2fcc66347a31cfe83f
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87848866"
---
# <a name="monitor-and-collect-data-from-ml-web-service-endpoints"></a>Supervisión y recopilación de datos de los puntos de conexión del servicio web ML
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo, aprenderá a recopilar datos y a realizar la supervisión de los modelos implementados en los puntos de conexión de servicio web en Azure Kubernetes Service (AKS) o Azure Container Instances (ACI), para lo cual consultará los registros y habilitará Azure Application Insights a través de: 
* [SDK de Python de Azure Machine Learning](#python)
* [Azure Machine Learning Studio](#studio) en https://ml.azure.com

Además de recopilar los datos de salida y la respuesta de un punto de conexión, puede supervisar:

* Tasas de solicitudes, tiempos de respuesta y tasas de error
* Tasas de dependencias, tiempos de respuesta y tasas de error
* Excepciones

[Más información sobre Azure Application Insights](../azure-monitor/app/app-insights-overview.md). 


## <a name="prerequisites"></a>Requisitos previos

* Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

* Un área de trabajo de Azure Machine Learning, un directorio local que contenga los scripts y el SDK de Azure Machine Learning para Python instalado. Para obtener información sobre cómo obtener estos requisitos previos, consulte [Procedimiento de configuración de un entorno de desarrollo](how-to-configure-environment.md).

* Un modelo de Machine Learning entrenado para implementarse en Azure Kubernetes Service (AKS) o Azure Container Instance (ACI). Si no tiene uno, consulte el tutorial [Entrenamiento de un modelo de clasificación de imágenes](tutorial-train-models-with-aml.md).

## <a name="query-logs-for-deployed-models"></a>Consulta de registros para modelos implementados

Para recuperar los registros de un servicio web implementado anteriormente, cargue el servicio y use la función `get_logs()`. Los registros pueden contener información detallada sobre los errores que se produjeron durante la implementación.

```python
from azureml.core.webservice import Webservice

# load existing web service
service = Webservice(name="service-name", workspace=ws)
logs = service.get_logs()
```

## <a name="web-service-metadata-and-response-data"></a>Metadatos de servicio web y datos de respuesta

> [!IMPORTANT]
> Azure Application Insights solo registra cargas de hasta 64 kb. Si se alcanza este límite, puede que vea errores como memoria insuficiente o que no se registre ninguna información.

Para registrar la información de una solicitud en el servicio web, agregue instrucciones `print` al archivo score.py. Cada instrucción `print` genera una entrada en la tabla de seguimientos en Application Insights, en el mensaje `STDOUT`. El contenido de la instrucción `print` se incluirá en `customDimensions` y, luego, `Contents` en la tabla de seguimientos. Si imprime una cadena JSON, se genera una estructura de datos jerárquica en el resultado de seguimientos, en `Contents`.

Puede consultar Azure Application Insights directamente para acceder a estos datos o configurar una [exportación continua](https://docs.microsoft.com/azure/azure-monitor/app/export-telemetry) a una cuenta de almacenamiento para una retención más larga o un procesamiento adicional. Los datos del modelo se pueden utilizar entonces en Azure Machine Learning para configurar el etiquetado, un nuevo entrenamiento, la capacidad de explicación, el análisis de datos u otro uso. 


<a name="python"></a>

## <a name="use-python-sdk-to-configure"></a>Uso del SDK de Python para configurar 

### <a name="update-a-deployed-service"></a>Actualización de un servicio implementado

1. Identifique el servicio en el área de trabajo. El valor de `ws` es el nombre del área de trabajo.

    ```python
    from azureml.core.webservice import Webservice
    aks_service= Webservice(ws, "my-service-name")
    ```
2. Actualice el servicio y habilite Azure Application Insights.

    ```python
    aks_service.update(enable_app_insights=True)
    ```

### <a name="log-custom-traces-in-your-service"></a>Registro de seguimientos personalizados del servicio

Si desea registrar seguimientos personalizados, siga el proceso de implementación estándar de AKS o ACI en el documento sobre [cómo realizar la implementación y donde](how-to-deploy-and-where.md). Después, siga estos pasos:

1. Para enviar datos a Application Insights durante la inferencia, actualice el archivo de puntuación agregando las instrucciones PRINT. Para registrar información más compleja, como los datos de la solicitud y la respuesta, use una estructura JSON. En el siguiente ejemplo de archivo score.py se registra la hora en que se inicializa el modelo, la entrada y la salida durante la inferencia, y la hora en que se produjeron los errores:

    > [!IMPORTANT]
    > Azure Application Insights solo registra cargas de hasta 64 kb. Si se alcanza este límite, puede que vea errores como memoria insuficiente o que no se registre ninguna información. Si los datos que desea registrar tienen más de 64 kB, debe almacenarlos en el almacenamiento de blobs usando la información de [Recopilar datos de modelos en producción](how-to-enable-data-collection.md).
    
    ```python
    import pickle
    import json
    import numpy 
    from sklearn.externals import joblib
    from sklearn.linear_model import Ridge
    from azureml.core.model import Model
    import time

    def init():
        global model
        #Print statement for appinsights custom traces:
        print ("model initialized" + time.strftime("%H:%M:%S"))
        
        # note here "sklearn_regression_model.pkl" is the name of the model registered under the workspace
        # this call should return the path to the model.pkl file on the local disk.
        model_path = Model.get_model_path(model_name = 'sklearn_regression_model.pkl')
        
        # deserialize the model file back into a sklearn model
        model = joblib.load(model_path)
    

    # note you can pass in multiple rows for scoring
    def run(raw_data):
        try:
            data = json.loads(raw_data)['data']
            data = numpy.array(data)
            result = model.predict(data)
            # Log the input and output data to appinsights:
            info = {
                "input": raw_data,
                "output": result.tolist()
                }
            print(json.dumps(info))
            # you can return any datatype as long as it is JSON-serializable
            return result.tolist()
        except Exception as e:
            error = str(e)
            print (error + time.strftime("%H:%M:%S"))
            return error
    ```

2. Actualice la configuración del servicio.
    
    ```python
    config = Webservice.deploy_configuration(enable_app_insights=True)
    ```

3. Cree una imagen e impleméntela en [AKS o ACI](how-to-deploy-and-where.md).

Para obtener más información sobre el registro y la recopilación de datos, vea [Habilitación del registro en Azure Machine Learning](how-to-enable-logging.md) y [Recopilación de datos de modelos en producción](how-to-enable-data-collection.md).

### <a name="disable-tracking-in-python"></a>Deshabilitación del seguimiento en Python

Para deshabilitar Azure Application Insights, use el siguiente código:

```python 
## replace <service_name> with the name of the web service
<service_name>.update(enable_app_insights=False)
```

<a name="studio"></a>

## <a name="use-azure-machine-learning-studio-to-configure"></a>Uso de Azure Machine Learning Studio para la configuración

También puede habilitar Azure Application Insights desde Azure Machine Learning Studio cuando esté listo para implementar el modelo con estos pasos.

1. Inicie sesión en su área de trabajo en https://ml.azure.com/.
1. Vaya a **Modelos** y seleccione el modelo que desea implementar.
1. Seleccione **+Implementar**.
1. Rellene el formulario **Implementar modelo**.
1. Expanda el menú **Opciones avanzadas**.

    ![Implementar modelo](./media/how-to-enable-app-insights/deploy-form.png)
1. Seleccione **Enable Application Insights diagnostics and data collection** (Habilitar la recopilación de datos y el diagnóstico de Application Insights).

    ![Habilitar Application Insights](./media/how-to-enable-app-insights/enable-app-insights.png)

## <a name="view-metrics-and-logs"></a>Visualización de métricas y registros

Los datos del servicio se almacenan en la cuenta de Azure Application Insights en el mismo grupo de recursos de Azure Machine Learning.
Para verlo:

1. Vaya al área de trabajo de Azure Machine Learning en [Studio](https://ml.azure.com/).
1. Seleccione **Puntos de conexión**.
1. Seleccione el servicio implementado.
1. Desplácese hacia abajo para buscar la **URL de Application Insights** y seleccione el vínculo.

    [![Localizar la dirección URL de Application Insights](./media/how-to-enable-app-insights/appinsightsloc.png)](././media/how-to-enable-app-insights/appinsightsloc.png#lightbox)

1. En Application Insights, en la pestaña **Información general** o en la sección __Supervisión__ de la lista de la izquierda, seleccione __Registros__.

    [![Pestaña Información general de supervisión](./media/how-to-enable-app-insights/overview.png)](./media/how-to-enable-app-insights/overview.png#lightbox)

1. Para ver la información registrada en el archivo score.py, examine la tabla de __seguimientos__. La siguiente consulta busca registros en los que se registró el valor de __entrada__:

    ```kusto
    traces
    | where customDimensions contains "input"
    | limit 10
    ```

   [![datos de seguimiento](./media/how-to-enable-app-insights/model-data-trace.png)](././media/how-to-enable-app-insights/model-data-trace.png#lightbox)

Para más información sobre el uso de Azure Application Insights, consulte [¿Qué es Application Insights?](../azure-monitor/app/app-insights-overview.md).

## <a name="export-data-for-further-processing-and-longer-retention"></a>Exportación de datos para su posterior procesamiento y una retención más duradera

>[!Important]
> Azure Application Insights solo admite exportaciones a Blob Storage. Otros límites de esta funcionalidad de exportación se enumeran en [Exportación de telemetría desde Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/export-telemetry#continuous-export-advanced-storage-configuration).

Puede usar la [exportación continua](https://docs.microsoft.com/azure/azure-monitor/app/export-telemetry) de Azure Application Insights para enviar mensajes a una cuenta de almacenamiento admitida, donde se pueda establecer una retención más larga. Los datos se almacenan en formato JSON y se pueden analizar fácilmente para extraer los datos del modelo. 

Se puede usar Azure Data Factory, canalizaciones de Azure Machine Learning u otras herramientas de procesamiento de datos para transformar los datos según sea necesario. Una vez transformados los datos, puede registrarlos en el área de trabajo de Azure Machine Learning como un conjunto de datos. Para ello, vea [Creación de conjuntos de datos y acceso a ellos (versión preliminar) en Azure Machine Learning](how-to-create-register-datasets.md).

:::image type="content" source="media/how-to-enable-app-insights/continuous-export-setup.png" alt-text="Exportación continua":::


## <a name="example-notebook"></a>Cuaderno de ejemplo

El cuaderno [enable-app-insights-in-production-service.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/enable-app-insights-in-production-service/enable-app-insights-in-production-service.ipynb) muestra los conceptos de este artículo. 
 
[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Implementación de un modelo en un clúster de Azure Kubernetes Service](https://docs.microsoft.com/azure/machine-learning/how-to-deploy-azure-kubernetes-service) o [Implementación de un modelo en Azure Container Instances](https://docs.microsoft.com/azure/machine-learning/how-to-deploy-azure-container-instance) para implementar los modelos en los puntos de conexión del servicio web y permitir que Azure Application Insights aproveche la recopilación de datos y la supervisión de puntos de conexión.
* Consulte [MLOps: Administración, implementación y supervisión de modelos con Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/concept-model-management-and-deployment) para más información sobre cómo aprovechar los datos recopilados de los modelos en producción. Estos datos pueden ayudar a mejorar continuamente el proceso de aprendizaje automático.
