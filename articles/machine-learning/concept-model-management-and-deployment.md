---
title: 'MLOps: Administración de modelos de Machine Learning'
titleSuffix: Azure Machine Learning
description: 'Más información sobre la administración de modelos con Azure Machine Learning (MLOps) Implemente, administre y supervise sus propios modelos para mejorarlos permanentemente. '
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
author: jpe316
ms.author: jordane
ms.date: 03/17/2020
ms.custom: seodec18
ms.openlocfilehash: d1d14fa9730e3ddd47378a45ff7a1442bdee69ac
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543391"
---
# <a name="mlops-model-management-deployment-and-monitoring-with-azure-machine-learning"></a>MLOps: Administración, implementación y supervisión de modelos con Azure Machine Learning

En este artículo, obtendrá información sobre cómo usar Azure Machine Learning para administrar el ciclo de vida de los modelos. Azure Machine Learning usa un enfoque de Operaciones de Machine Learning (MLOps). MLOps mejora la calidad y la coherencia de las soluciones de aprendizaje automático. 

## <a name="what-is-mlops"></a>¿Qué es MLOps?

Operaciones de Machine Learning (MLOps) se basa en los principios y prácticas de [DevOps](https://azure.microsoft.com/overview/what-is-devops/) que aumentan la eficacia de los flujos de trabajo. Por ejemplo, integración continua, entrega e implementación. MLOps aplica estos principios al proceso de aprendizaje automático, con el objetivo de:

* Conseguir una experimentación y desarrollo de modelos más rápidos
* Conseguir una implementación más rápida de los modelos en producción
* Control de calidad

Azure Machine Learning ofrece las siguientes funcionalidades de MLOps:

- **Creación de canalizaciones de ML reproducibles**. Las canalizaciones de Machine Learning permiten definir pasos repetibles y reutilizables para los procesos de preparación de datos, entrenamiento y puntuación.
- **Cree entornos de software reutilizables** para entrenar e implementar modelos.
- **Registro, empaquetado e implementación de modelos desde cualquier lugar**. También puede realizar el seguimiento de los metadatos asociados necesarios para utilizar el modelo.
- **Captura de los datos de gobernanza del ciclo de vida de Machine Learning de un extremo a otro**. La información registrada puede incluir quién está publicando modelos, por qué se han realizado los cambios y cuándo se implementaron o usaron los modelos en producción.
- **Notificación y alerta sobre eventos en el ciclo de vida de Machine Learning**. Por ejemplo, la finalización del experimento, el registro del modelo, la implementación de este y la detección del desfase de datos.
- **Supervisión de las aplicaciones de ML para las incidencias relacionadas con ML y operativas**. Compare las entradas del modelo durante el entrenamiento y la inferencia, explore las métricas de un modelo específico e incluya supervisión y alertas en su infraestructura de ML.
- **Automatización del ciclo de vida de un extremo a otro de Machine Learning con Azure Machine Learning y Azure Pipelines**. El uso de canalizaciones le permite actualizar con frecuencia los modelos, probar los modelos nuevos e implementar continuamente nuevos modelos de ML junto con sus otras aplicaciones y servicios.

## <a name="create-reproducible-ml-pipelines"></a>Creación de canalizaciones de ML reproducibles

Utilice canalizaciones de ML desde Azure Machine Learning para unir todos los pasos implicados en el proceso de entrenamiento del modelo.

Una canalización de ML puede contener pasos que van desde la preparación de datos hasta la extracción de características, el ajuste de hiperparámetros o la evaluación del modelo. Para obtener más información, consulte el artículo [canalizaciones de aprendizaje automático](concept-ml-pipelines.md).

Si utiliza el [Diseñador](concept-designer.md) para crear sus canalizaciones ML, en cualquier momento puede hacer clic en **"..."** en la parte superior derecha de la página del Diseñador y luego seleccionar **Clonar**. La clonación de la canalización le permite iterar el diseño de canalización sin perder las versiones anteriores.  

## <a name="create-reusable-software-environments"></a>Creación de entornos de software reutilizables

Use los entornos de Azure Machine Learning para realizar un seguimiento de las dependencias de software de sus proyectos y reproducirlas a medida que evolucionan. Los entornos le permiten asegurarse de que las compilaciones se pueden reproducir sin necesidad de configuración manual del software.

Los entornos describen las dependencias PIP y Conda de los proyectos, y se pueden usar para el entrenamiento e implementación de modelos. Para más información, consulte [¿Qué son los entornos de Azure Machine Learning?](concept-environments.md).

## <a name="register-package-and-deploy-models-from-anywhere"></a>Registro, empaquetado e implementación de modelos desde cualquier lugar

### <a name="register-and-track-ml-models"></a>Registro y supervisión de modelos de aprendizaje automático

El registro del modelo permite almacenar y versionar los modelos en la nube de Azure, en su área de trabajo. El registro de modelo facilita organizar y mantener un seguimiento de los modelos entrenados.

> [!TIP]
> Un modelo registrado es un contenedor lógico para uno o varios archivos que componen el modelo. Por ejemplo, si tiene un modelo que se almacena en varios archivos, puede registrarlos como un único modelo en el área de trabajo de Azure Machine Learning. Después del registro, puede descargar o implementar el modelo registrado y recibir todos los archivos que se registraron.

Los modelos registrados se identifican por el nombre y la versión. Cada vez que registra un modelo con el mismo nombre que uno existente, el registro incrementa la versión. Se pueden proporcionar etiquetas de metadatos adicionales durante el registro. Después, estas etiquetas se usan al buscar un modelo. Azure Machine Learning admite cualquier modelo que pueda cargarse mediante Python 3.5.2 o posterior.

> [!TIP]
> También puede registrar modelos entrenados fuera de Azure Machine Learning.

No se puede eliminar un modelo registrado que se esté usando en una implementación activa.
Para más información, consulte la sección de registro de modelos de [Implementación de modelos](how-to-deploy-and-where.md#registermodel).

### <a name="profile-models"></a>Modelos de perfil

Azure Machine Learning puede ayudarle a comprender los requisitos de CPU y memoria del servicio que se creará al implementar el modelo. La generación de perfiles prueba el servicio que ejecuta el modelo y devuelve información como el uso de la CPU, el uso de memoria y la latencia de respuesta. También proporciona una recomendación de CPU y memoria basada en el uso de recursos.
Para más información, consulte la sección de generación de perfiles de [Implementación de modelos](how-to-deploy-profile-model.md).

### <a name="package-and-debug-models"></a>Empaquetado y depuración de modelos

Antes de implementar un modelo en producción, se empaqueta en una imagen de Docker. En la mayoría de los casos, la creación de imágenes se produce automáticamente en segundo plano durante la implementación. Se puede especificar la imagen manualmente.

Si experimenta problemas con la implementación, puede implementarla en su entorno de desarrollo local para solucionar problemas y depurarla.

Para obtener más información, consulte [implementación de modelos](how-to-deploy-and-where.md#registermodel) y [solución problemas en implementaciones](how-to-troubleshoot-deployment.md).

### <a name="convert-and-optimize-models"></a>Conversión y optimización de modelos

Si convierte el modelo a [Open Neural Network Exchange](https://onnx.ai) (ONNX), puede mejorar el rendimiento. En promedio, la conversión a ONNX puede duplicar el rendimiento.

Para más información sobre ONNX con Azure Machine Learning, consulte el artículo sobre [creación y aceleración de los modelos de aprendizaje automático](concept-onnx.md).

### <a name="use-models"></a>Uso de modelos

Los modelos de Machine Learning entrenados se implementan como servicios web en la nube o localmente. También puede implementar en dispositivos Azure IoT Edge. Las implementaciones utilizan matrices de puertas programable (FPGA) por CPU, GPU o campos para las inferencias. También puede usar modelos de Power BI.

Cuando use un modelo como servicio web o dispositivo de IoT Edge, proporcione los siguientes elementos:

* Los modelos que se usan para puntuar los datos enviados al servicio o dispositivo.
* Un script de entrada. Este script acepta las solicitudes, usa los modelos para puntuar los datos y devuelve una respuesta.
* Un entorno de Azure Machine Learning que describe las dependencias de PIP y Conda que requieren los modelos y el script de entrada.
* Los recursos adicionales, como texto, datos, etc. requeridos por el script de entrada y los modelos.

También debe proporcionar la configuración de la plataforma de implementación de destino. Por ejemplo, el tipo de familia de la máquina virtual, la memoria disponible y número de núcleos cuando se implemente en Azure Kubernetes Service.

Cuando se crea la imagen, también se agregan los componentes requeridos por Azure Machine Learning. Por ejemplo, los recursos necesarios para ejecutar el servicio web e interactuar con IoT Edge.

#### <a name="batch-scoring"></a>Puntuación por lotes
La puntuación por lotes son compatibles con las canalizaciones de ML. Para obtener más información, vea [Predicciones por lotes de macrodatos](how-to-use-parallel-run-step.md).

#### <a name="real-time-web-services"></a>Servicios web en tiempo real

Puede usar los modelos en **servicios web** con los siguientes destinos de proceso:

* Azure Container Instances
* Azure Kubernetes Service
* Entorno de desarrollo locales

Para implementar el modelo como un servicio web, debe proporcionar los siguientes elementos:

* El modelo o un conjunto de modelos.
* Las dependencias necesarias para usar el modelo. Por ejemplo, un script que acepta solicitudes e invoca el modelo, las dependencias de conda, etcétera.
* La configuración de implementación que describe cómo y dónde implementar el modelo.

Para obtener más información, consulte [Implementación de modelos](how-to-deploy-and-where.md).

#### <a name="controlled-rollout"></a>Lanzamiento controlado

Al implementar en Azure Kubernetes Service, puede usar el lanzamiento controlado para habilitar los siguientes escenarios:

* Creación de varias versiones de un punto de conexión para una implementación
* Realización de pruebas A/B mediante el enrutamiento del tráfico a diferentes versiones del punto de conexión
* Cambio entre versiones del punto de conexión actualizando el porcentaje de tráfico en la configuración del punto de conexión.

Para más información, consulte [Lanzamiento controlado de modelos de ML](how-to-deploy-azure-kubernetes-service.md#deploy-models-to-aks-using-controlled-rollout-preview).

#### <a name="iot-edge-devices"></a>Dispositivos de IoT Edge

Puede usar modelos con dispositivos IoT a través de los **módulos de Azure IoT Edge**. Los módulos de IoT Edge se implementan en un dispositivo de hardware, lo que permite la inferencia o puntuación de modelos en el dispositivo.

Para obtener más información, consulte [Implementación de modelos](how-to-deploy-and-where.md).

### <a name="analytics"></a>Análisis

Microsoft Power BI admite el uso de modelos de Machine Learning para el análisis de datos. Para obtener más información, consulte la [integración de Azure Machine Learning en Power BI (versión preliminar)](https://docs.microsoft.com/power-bi/service-machine-learning-integration).

## <a name="capture-the-governance-data-required-for-capturing-the-end-to-end-ml-lifecycle"></a>Captura de los datos de gobernanza necesarios para capturar el ciclo de vida de ML de un extremo a otro

Azure Machine Learning ofrece la capacidad de realizar un seguimiento del registro de auditoría de un extremo a otro de todos los recursos de ML mediante metadatos.

- Azure Machine Learning [se integra con Git](how-to-set-up-training-targets.md#gitintegration) para realizar un seguimiento de la información sobre el repositorio, rama o commit del que procede el código.
- Los [conjuntos de datos de Azure Machine Learning](how-to-create-register-datasets.md) ayudan a realizar un seguimiento, generar perfiles y realizar versiones de sus datos.
- La [interpretabilidad](how-to-machine-learning-interpretability.md) permite explicar los modelos, satisfacer el cumplimiento normativo y comprender cómo llegan los modelos a un resultado para la entrada determinada.
- El historial de ejecución de Azure Machine Learning almacena una instantánea del código, los datos y los procesos utilizados para entrenar un modelo.
- El registro de modelos de Azure Machine Learning captura todos los metadatos asociados al modelo (el experimento que lo entrenó, dónde se está implementando, si las implementaciones son correctas).
- La [integración con Azure](how-to-use-event-grid.md) permite actuar en los eventos del ciclo de vida de Machine Learning. Por ejemplo, el registro de modelos, la implementación, el desfase de datos y los eventos de aprendizaje (ejecución).

> [!TIP]
> Aunque parte de la información sobre los modelos y conjuntos de datos se capturan automáticamente, puede agregar información adicional mediante __etiquetas__. Al buscar modelos y conjuntos de datos registrados en el área de trabajo, puede usar etiquetas como filtro.
>
> Asociar un conjunto de datos a un modelo registrado es un paso opcional. Para obtener información sobre cómo hacer referencia a un conjunto de datos al registrar un modelo, vea la referencia de clase [Modelo](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model(class)?view=azure-ml-py).


## <a name="notify-automate-and-alert-on-events-in-the-ml-lifecycle"></a>Notificación, automatización y alerta sobre eventos en el ciclo de vida de ML
Azure Machine Learning publica eventos clave en Azure EventGrid, que se puede usar para notificar y automatizar eventos en el ciclo de vida de ML. Para obtener más información, vea [este documento](how-to-use-event-grid.md).


## <a name="monitor-for-operational--ml-issues"></a>Supervisión de problemas operativos y de Machine Learning

La supervisión permite comprender qué datos se envían a su modelo y las predicciones que devuelve.

Esta información le ayudará a comprender cómo se usa el modelo. Los datos de entrada recopilados también pueden ser útiles para entrenar versiones futuras del modelo.

Para más información, consulte [cómo habilitar la recopilación de datos de un modelo](how-to-enable-data-collection.md).

## <a name="retrain-your-model-on-new-data"></a>Nuevo entrenamiento del modelo con datos nuevos

A menudo, querrá validar el modelo, actualizarlo o incluso volver a entrenarlo desde cero, a medida que reciba información nueva. A veces, la recepción de nuevos datos es una parte esperada del dominio. Otras veces, como se explica en [Detección del desfase de datos (versión preliminar) en los conjuntos de datos](how-to-monitor-datasets.md), el rendimiento del modelo puede reducirse en función de los cambios realizados en un sensor determinado, los cambios naturales en los datos, como los efectos estacionales, o las características que se desplazan en relación con otras características. 

No hay ninguna respuesta universal a la pregunta "Cómo puedo saber si debo volver a entrenarlo?" Sin embargo, las herramientas de supervisión y eventos de Azure ML descritas anteriormente son buenos puntos de partida para la automatización. Una vez que haya decidido volver a entrenarlo, debe: 

- Realizar un preprocesamiento de los datos mediante un proceso repetible y automatizado
- Entrenar el nuevo modelo
- Comparar las salidas del nuevo modelo con las del modelo anterior
- Usar criterios predefinidos para elegir si quiere reemplazar el modelo anterior 

Un tema a considerar de los pasos anteriores es que el reciclaje debe estar automatizado, en lugar de ser ad hoc. Las [canalizaciones de Azure Machine Learning](concept-ml-pipelines.md) son una buena respuesta para crear flujos de trabajo relacionados con la preparación, el entrenamiento, la validación y la implementación de los datos. Lea [Volver a entrenar modelos con el diseñador de Azure Machine Learning](how-to-retrain-designer.md) para ver cómo se ajustan las canalizaciones y el diseñador de Azure Machine Learning en un escenario de reentrenamiento. 

## <a name="automate-the-ml-lifecycle"></a>Automatización del ciclo de vida de Machine Learning 

Puede usar GitHub y Azure Pipelines para crear un proceso de integración continua que entrene un modelo. En un escenario común, cuando un científico de datos comprueba un cambio en el repositorio de Git para un proyecto, Azure Pipelines inicia una ejecución de entrenamiento. A continuación, se pueden inspeccionar los resultados de la ejecución para ver las características de rendimiento del modelo entrenado. También puede crear una canalización que implemente el modelo como un servicio web.

La [extensión de Azure Machine Learning](https://marketplace.visualstudio.com/items?itemName=ms-air-aiagility.vss-services-azureml) facilita el trabajo con Azure Pipelines. Ofrece las siguientes mejoras a Azure Pipelines:

* Habilita la selección de áreas de trabajo al definir una conexión de servicio.
* Permite que los modelos entrenados creados con una canalización de entrenamiento desencadenen las canalizaciones de versión.

Para obtener más información sobre el uso de Azure Pipelines con Azure Machine Learning, consulte los vínculos siguientes:

* [Integración continua e implementación de modelos de Machine Learning con Azure Pipelines](/azure/devops/pipelines/targets/azure-machine-learning). 
* Repositorio de [MLOPs de Azure Machine Learning](https://aka.ms/mlops).
* Repositorio de [MLOPs de Python en Azure Machine Learning](https://github.com/Microsoft/MLOpspython).

También puede usar Azure Data Factory para crear una canalización de ingesta de datos que prepare los datos para su uso con el entrenamiento. Para más información, consulte [Canalización de ingesta de datos](how-to-cicd-data-ingestion.md).

## <a name="next-steps"></a>Pasos siguientes

Para más información, lea y explore los siguientes recursos:

+ [Cómo y dónde sed implementan los modelos](how-to-deploy-and-where.md) con Azure Machine Learning

+ [Tutorial: Implementación de un modelo de clasificación de imágenes en ACI](tutorial-deploy-models-with-aml.md).

+ [Repositorio de ejemplos de MLOps de un extremo a otro](https://github.com/microsoft/MLOps)

+ [CI/CD de modelos de Machine Learning con Azure Pipelines](/azure/devops/pipelines/targets/azure-machine-learning)

+ Creación de clientes que [consumen un modelo implementado](how-to-consume-web-service.md)

+ [Aprendizaje automático a escala](/azure/architecture/data-guide/big-data/machine-learning-at-scale)

+ [Repositorio de procedimientos recomendados y arquitecturas de referencia de Azure AI](https://github.com/microsoft/AI)
