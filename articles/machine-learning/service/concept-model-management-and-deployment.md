---
title: 'MLOps: Administrar, implementar y supervisar los modelos de aprendizaje automático'
titleSuffix: Azure Machine Learning service
description: 'Aprenda a usar el servicio Azure Machine Learning para MLOps: implementar, administrar y supervisar sus propios modelos para mejorarlos continuamente. Puede implementar los modelos que haya entrenado con Azure Machine Learning Services, en la máquina local o que procedan de otros orígenes.'
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
author: chris-lauren
ms.author: clauren
ms.date: 05/02/2019
ms.custom: seodec18
ms.openlocfilehash: d02318db4deae6f0a6acdae9382ce47116c95110
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65800870"
---
# <a name="mlops-manage-deploy-and-monitor-models-with-azure-machine-learning-service"></a>MLOps: Administración, implementación y supervisión de modelos Azure Machine Learning Services

En este artículo aprenderá a usar Azure Machine Learning Services para implementar, administrar y supervisar sus propios modelos para mejorarlos continuamente. Puede implementar los modelos que haya entrenado con Azure Machine Learning, en la máquina local o que procedan de otros orígenes. 

En el siguiente diagrama se ilustra el flujo de trabajo de implementación completo: [![Flujo de trabajo de implementación para Azure Machine Learning](media/concept-model-management-and-deployment/deployment-pipeline.png)](media/concept-model-management-and-deployment/deployment-pipeline.png#lightbox)

El MLOps / flujo de trabajo de implementación incluye los pasos siguientes:
1. **Registrar el modelo** en un registro hospedado en el área de trabajo de Azure Machine Learning Services
1. **Use** el modelo en un servicio web en la nube, en un dispositivo de IoT, o para realizar análisis con Power BI.
1. **Supervisar y recopilar datos**
1. **Actualizar** una implementación para usar una imagen nueva.

Cada paso puede realizarse de forma independiente o como parte de un único comando. Además, puede crear un **flujo de trabajo de CI/CD** tal como se muestra en este gráfico.

[!['Ciclo de implementación (CI/CD) continua e integración continua de azure Machine Learning'](media/concept-model-management-and-deployment/model-ci-cd.png)](media/concept-model-management-and-deployment/model-ci-cd.png#lightbox)

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE2X1GX]

## <a name="step-1-register-model"></a>Paso 1: Registro del modelo

El registro del modelo permite almacenar y versionar los modelos en la nube de Azure, en su área de trabajo. El registro de modelo facilita organizar y mantener un seguimiento de los modelos entrenados.

> [!TIP]
> También puede registrar modelos entrenados fuera del servicio de Azure Machine Learning.
 
Los modelos registrados se identifican por el nombre y la versión. Cada vez que registra un modelo con el mismo nombre que uno existente, el registro incrementa la versión. También puede proporciona etiquetas de metadatos adicionales durante el registro para usarlas al buscar modelos. El servicio de Azure Machine Learning es compatible con cualquier modelo que se puede cargar mediante Python 3.5.2 o superior.

No se puede eliminar los modelos que se usan en una implementación activa.

Para más información, consulte la sección de registro de modelos de [Implementación de modelos](how-to-deploy-and-where.md#registermodel).

Para un ejemplo de registro de un modelo almacenado en formato pickle, consulte [Tutorial: Entrenamiento de un modelo de clasificación de imágenes](tutorial-deploy-models-with-aml.md).

## <a name="step-2-use-the-model"></a>Paso 2: Usar el modelo

Modelos de Machine learning pueden usarse como un servicio web, en dispositivos IoT Edge, o para realizar análisis de servicios como Power BI.

### <a name="web-service"></a>Servicio Web

Puede usar los modelos en **servicios web** con los siguientes destinos de proceso:

* Instancia de Azure Container
* Azure Kubernetes Service

Para implementar el modelo como un servicio web, debe proporcionar lo siguiente:

* El modelo o un conjunto de modelos.
* Dependencias necesarias para utilizar el modelo. Por ejemplo, una secuencia de comandos que acepta solicitudes e invoca el modelo, las dependencias de conda, etcetera.
* Configuración de implementación que se describe cómo y dónde para implementar el modelo.

Para obtener más información, consulte [implementar modelos](how-to-deploy-and-where.md).

### <a name="iot-edge-devices"></a>Dispositivos IoT Edge

Puede usar modelos con dispositivos IoT a través de **módulos de Azure IoT Edge**. Módulos de IoT Edge se implementan en dispositivos de hardware, lo que permite la inferencia en el dispositivo.

Para obtener más información, consulte [implementar modelos](how-to-deploy-and-where.md).

### <a name="analytics"></a>Análisis

Microsoft Power BI admite el uso de modelos de aprendizaje automático para el análisis de datos. Para obtener más información, consulte [integración de Azure Machine Learning en Power BI (versión preliminar)](https://docs.microsoft.com/power-bi/service-machine-learning-integration).

## <a name="step-3-monitor-models-and-collect-data"></a>Paso 3: Supervisión de los modelos y recopilación de datos

Supervisión permite comprender qué datos se envían a su modelo y las predicciones que devuelve.

Esta información le ayudará a comprender cómo se usa el modelo. Los datos recopilados de entrada también pueden ser útiles en versiones futuras de entrenamiento del modelo.

Para más información, consulte [cómo habilitar la recopilación de datos de un modelo](how-to-enable-data-collection.md).

## <a name="step-4-update-the-deployment"></a>Paso 4: Actualización de la implementación

Las implementaciones se deben actualizar explícitamente. Para más información, consulte la sección de actualización de [Implementación de modelos](how-to-deploy-and-where.md#update).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre [cómo y dónde puede implementar modelos](how-to-deploy-and-where.md) con el servicio Azure Machine Learning. Para obtener un ejemplo de implementación, consulte [Tutorial: Implementar un modelo de clasificación de imágenes en Azure Container Instances](tutorial-deploy-models-with-aml.md).

Aprenda a crear [integración continua e implementación de modelos de aprendizaje automático con canalizaciones de Azure](/azure/devops/pipelines/targets/azure-machine-learning). 

Obtenga información sobre cómo crear aplicaciones de cliente y servicios que [consumen un modelo implementado como servicio web](how-to-consume-web-service.md).
