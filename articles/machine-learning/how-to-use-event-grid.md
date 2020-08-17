---
title: Desencadenar eventos en flujos de trabajo de aprendizaje automático (versión preliminar)
titleSuffix: Azure Machine Learning
description: Configure aplicaciones basadas en eventos, procesos o flujos de trabajo de aprendizaje automático de CI/CD en Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: shipatel
author: shivp950
ms.reviewer: larryfr
ms.date: 05/11/2020
ms.openlocfilehash: 3b6f115e27b5cf956a524bd1a610797c689b5d6c
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319497"
---
# <a name="trigger-applications-processes-or-cicd-workflows-based-on-azure-machine-learning-events-preview"></a>Desencadenar aplicaciones, procesos o flujos de trabajo de CI/CD basados en eventos de Azure Machine Learning (versión preliminar)

En este artículo, aprenderá a configurar aplicaciones, procesos o flujos de trabajo de CI/CD basados en eventos de Azure Machine Learning, como correos electrónicos de notificación de errores o ejecuciones de canalización de ML, cuando se detecten determinadas condiciones en [Azure Event Grid](https://docs.microsoft.com/azure/event-grid/).

Azure Machine Learning administra todo el ciclo de vida del proceso de aprendizaje automático, incluido el entrenamiento del modelo, la implementación de modelo y la supervisión. Puede usar Event Grid para reaccionar a eventos de Azure Machine Learning, como la finalización de las ejecuciones de entrenamiento, el registro y la implementación de modelos, y la detección de desfases de datos mediante arquitecturas modernas sin servidor. Después, puede suscribirse y consumir eventos como el cambio en el estado de ejecución, la finalización de la ejecución, el registro de modelos, la implementación de modelos y la detección de desfase de datos dentro de un área de trabajo.

Cuándo usar Event Grid para las acciones controladas por eventos:
* Envío de correos electrónicos en caso de error de ejecución y al finalizar la ejecución
* Uso de una función de Azure una vez registrado un modelo
* Streaming de eventos desde Azure Machine Learning a varios puntos de conexión
* Desencadenamiento de una canalización de Machine Learning cuando se detecta un desfase

> [!NOTE] 
> Actualmente, los eventos runStatusChanged solo se desencadenan cuando el estado de ejecución es **failed**

## <a name="prerequisites"></a>Requisitos previos
Para usar Event Grid, necesita acceso de colaborador o propietario al área de trabajo de Azure Machine Learning para la que se crearán los eventos.

## <a name="the-event-model--types"></a>Modelo de evento y tipos

Azure Event Grid le eventos de orígenes tales como Azure Machine Learning y otros servicios de Azure. Estos eventos se envían después a controladores de eventos tales como Azure Event Hubs, Azure Functions, Logic Apps y otros. En el diagrama siguiente se muestra cómo se conectan los orígenes y los controladores en Event Grid, pero no proporciona una lista completa de las integraciones admitidas.

![Modelo funcional de Azure Event Grid](./media/concept-event-grid-integration/azure-event-grid-functional-model.png)

Para obtener más información sobre los orígenes de eventos y los controladores de eventos, consulte [¿Qué es Event Grid?](/azure/event-grid/overview).

### <a name="event-types-for-azure-machine-learning"></a>Tipos de eventos de Azure Machine Learning

Azure Machine Learning proporciona eventos en los distintos puntos del ciclo de vida de aprendizaje automático: 

| Tipo de evento | Descripción |
| ---------- | ----------- |
| `Microsoft.MachineLearningServices.RunCompleted` | Se genera cuando se completa la ejecución de un experimento de aprendizaje automático. |
| `Microsoft.MachineLearningServices.ModelRegistered` | Se genera cuando se registra un modelo de aprendizaje automático en el área de trabajo |
| `Microsoft.MachineLearningServices.ModelDeployed` | Se genera cuando se completa una implementación de servicio de inferencia con uno o varios modelos. |
| `Microsoft.MachineLearningServices.DatasetDriftDetected` | Se genera cuando se completa un trabajo de detección de desfase de datos para dos conjuntos de datos |
| `Microsoft.MachineLearningServices.RunStatusChanged` | Se genera cuando cambia un estado de ejecución, que actualmente solo se produce cuando un estado de ejecución es "incorrecto". |

### <a name="filter--subscribe-to-events"></a>Filtrar y suscribirse a eventos

Estos eventos se publican a través de Azure Event Grid. Con Azure Portal, PowerShell o la CLI de Azure, los clientes pueden suscribirse a eventos fácilmente [especificando uno o varios tipos de eventos y las condiciones de filtrado](/azure/event-grid/event-filtering). 

Al configurar eventos, puede aplicar filtros para que se desencadenen únicamente con datos de eventos específicos. En el ejemplo siguiente, para eventos de cambio de estado de ejecución, puede filtrar por tipos de ejecución. El evento solo se desencadena cuando se cumplen los criterios. Consulte el [Esquema de Event Grid de Azure Machine Learning](/azure/event-grid/event-schema-machine-learning) para obtener información sobre los datos de eventos que puede filtrar. 

Las suscripciones para eventos de Azure Machine Learning están protegidas por el control de acceso basado en rol (RBAC). Solo un [colaborador o propietario](how-to-assign-roles.md#default-roles) de un área de trabajo puede crear, actualizar y eliminar suscripciones a eventos.  Los filtros pueden aplicarse a las suscripciones de eventos, ya sea durante la [creación](/cli/azure/eventgrid/event-subscription?view=azure-cli-latest) de la suscripción de eventos o en un momento posterior. 


1. Vaya a Azure Portal y seleccione una suscripción nueva o existente. 

1. Seleccione la pestaña de filtros y desplácese hacia abajo hasta los filtros avanzados. En **Clave** y **Valor**, proporcione los tipos de propiedad por los que desee filtrar. Aquí puede ver que el evento solo se desencadenará cuando el tipo de ejecución sea una ejecución de canalización o de paso de canalización.  

    :::image type="content" source="media/how-to-use-event-grid/select-event-filters.png" alt-text="filtrado de eventos":::


+ **Filtrado por tipo de evento:** Una suscripción de eventos puede especificar uno o varios tipos de evento de Azure Machine Learning.

+ **Filtrado por asunto del evento:** Azure Event Grid admite filtros de asunto en función de las coincidencias de __comienza por__ y __termina en__, con el fin de que los eventos con un asunto coincidente se entreguen al suscriptor. Distintos eventos de aprendizaje automático tienen un formato de asunto diferente.

  | Tipo de evento | Formato de asunto | Asunto de ejemplo |
  | ---------- | ----------- | ----------- |
  | `Microsoft.MachineLearningServices.RunCompleted` | `experiments/{ExperimentId}/runs/{RunId}` | `experiments/b1d7966c-f73a-4c68-b846-992ace89551f/runs/my_exp1_1554835758_38dbaa94` |
  | `Microsoft.MachineLearningServices.ModelRegistered` | `models/{modelName}:{modelVersion}` | `models/sklearn_regression_model:3` |
  | `Microsoft.MachineLearningServices.ModelDeployed` | `endpoints/{serviceId}` | `endpoints/my_sklearn_aks` |
  | `Microsoft.MachineLearningServices.DatasetDriftDetected` | `datadrift/{data.DataDriftId}/run/{data.RunId}` | `datadrift/4e694bf5-712e-4e40-b06a-d2a2755212d4/run/my_driftrun1_1550564444_fbbcdc0f` |
  | `Microsoft.MachineLearningServices.RunStatusChanged` | `experiments/{ExperimentId}/runs/{RunId}` | `experiments/b1d7966c-f73a-4c68-b846-992ace89551f/runs/my_exp1_1554835758_38dbaa94` | 

+ **Filtrado avanzado**: Azure Event Grid también admite el filtrado avanzado basado en el esquema de eventos publicados. Los detalles del esquema de eventos de Azure Machine Learning se pueden encontrar en [Esquema de eventos de Azure Event Grid para Azure Machine Learning](../event-grid/event-schema-machine-learning.md).  Entre los filtrados avanzados de ejemplo que puede realizar se incluyen:

  En el caso del evento `Microsoft.MachineLearningServices.ModelRegistered`, filtrar el valor de etiqueta del modelo:

  ```
  --advanced-filter data.ModelTags.key1 StringIn ('value1')
  ```

  Para más información sobre cómo aplicar filtros, consulte el artículo de [Filtrado de eventos para Event Grid](https://docs.microsoft.com/azure/event-grid/how-to-filter-events).

## <a name="consume-machine-learning-events"></a>Consumo de eventos de Machine Learning

Las aplicaciones que controlan los eventos de Machine Learning deben seguir algunos procedimientos recomendados:

> [!div class="checklist"]
> * Dado que se pueden configurar varias suscripciones para enrutar eventos al mismo controlador, es importante no asumir que los eventos provienen de un origen determinado, sino comprobar el tema del mensaje para asegurarse de que proviene del área de trabajo de aprendizaje automático prevista.
> * De igual forma, compruebe que eventType es uno de los que está preparado para procesar y no asuma que todos los eventos que reciba van a ser los tipos que espera.
> * Dado que los mensajes pueden llegar desordenados y con cierto retraso, utilice los campos de etag para saber si la información acerca de los objetos sigue estando actualizada.  Además, utilice los campos del secuenciador para conocer el orden de los eventos en cualquier objeto determinado.
> * Ignore los campos que no comprenda. Esta práctica le ayudará a mantenerse resistente a las nuevas características que puedan agregarse en el futuro.
> * Las operaciones de Azure Machine Learning con errores o canceladas no desencadenarán un evento. Por ejemplo, si se produce un error en la implementación de un modelo, no se desencadenará Microsoft.MachineLearningServices.ModelDeployed. Tenga en cuenta este modo de error al diseñar las aplicaciones. Siempre puede usar el SDK de Azure Machine Learning, la CLI o el portal para comprobar el estado de una operación y comprender los motivos de error detallados.

Azure Event Grid permite a los clientes compilar controladores de mensajes desacoplados, que se pueden desencadenar mediante eventos de Azure Machine Learning. Algunos ejemplos importantes de controladores de mensajes son:
* Azure Functions
* Azure Logic Apps
* Azure Event Hubs
* Canalización de Azure Data Factory
* Webhooks genéricos, que se pueden hospedar en la plataforma Azure o en otro lugar

## <a name="set-up-in-azure-portal"></a>Configuración en Azure Portal

1. Abra [Azure Portal](https://portal.azure.com) y vaya al área de trabajo de Azure Machine Learning.

1. En la barra de la izquierda, seleccione __Eventos__ y después **Suscripciones a eventos**. 

    ![select-events-in-workspace.png](./media/how-to-use-event-grid/select-event.png)

1. Seleccione el tipo de evento que se va a consumir. Por ejemplo, en la captura de pantalla siguiente se ha seleccionado __Modelo registrado__, __Modelo implementado__, __Ejecución completada__ y __Desfase del conjunto de datos detectado__:

    ![add-event-type](./media/how-to-use-event-grid/add-event-type-updated.png)

1. Seleccione el punto de conexión en el que se va a publicar el evento. En la captura de pantalla siguiente, __Centro de eventos__ es el punto de conexión seleccionado:

    ![select-event-handler](./media/how-to-use-event-grid/select-event-handler.png)

Una vez que haya confirmado la selección, haga clic en __Crear__. Después de la configuración, estos eventos se insertarán en el punto de conexión.


### <a name="set-up-with-the-cli"></a>Configuración con la CLI

Puede instalar la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) más reciente, o bien usar Azure Cloud Shell que se proporciona como parte de la suscripción de Azure.

Para instalar la extensión Event Grid, use el siguiente comando de la CLI:

```azurecli-interactive
az add extension --name eventgrid
```

En el ejemplo siguiente se muestra cómo seleccionar una suscripción de Azure y, después, crear una suscripción a eventos para Azure Machine Learning:

```azurecli-interactive
# Select the Azure subscription that contains the workspace
az account set --subscription "<name or ID of the subscription>"

# Subscribe to the machine learning workspace. This example uses EventHub as a destination. 
az eventgrid event-subscription create --name {eventGridFilterName} \
  --source-resource-id /subscriptions/{subId}/resourceGroups/{RG}/providers/Microsoft.MachineLearningServices/workspaces/{wsName} \
  --endpoint-type eventhub \
  --endpoint /subscriptions/{SubID}/resourceGroups/TestRG/providers/Microsoft.EventHub/namespaces/n1/eventhubs/EH1 \
  --included-event-types Microsoft.MachineLearningServices.ModelRegistered \
  --subject-begins-with "models/mymodelname"
```

## <a name="examples"></a>Ejemplos

### <a name="example-send-email-alerts"></a>Ejemplo: Envío de alertas de correo electrónico

Use [Azure Logic Apps](https://docs.microsoft.com/azure/logic-apps/) para configurar mensajes de correo electrónico para todos los eventos. Personalice con condiciones y especifique destinatarios para permitir la colaboración y el reconocimiento entre los equipos que trabajan juntos.

1. En Azure Portal, vaya al área de trabajo Azure Machine Learning y seleccione la pestaña Eventos en la barra de la izquierda. Desde ahí, seleccione __Aplicaciones lógicas__. 

    ![select-logic-ap](./media/how-to-use-event-grid/select-logic-ap.png)

1. Inicie sesión en la interfaz de usuario de la aplicación lógica y seleccione Machine Learning Service como el tipo de tema. 

    ![select-topic-type](./media/how-to-use-event-grid/select-topic-type.png)

1. Seleccione los eventos que quiere que le notifiquen. Por ejemplo, en la captura de pantalla siguiente, __RunCompleted__.

    ![select-event-runcomplete](./media/how-to-use-event-grid/select-event-runcomplete.png)

1. Puede usar el método de filtrado de la sección anterior o agregar filtros para desencadenar solo la aplicación lógica en un subconjunto de tipos de evento. En la captura de pantalla siguiente, se usa un __filtro de prefijo__ de __/datadriftID/runs/__ .

    ![filter-events](./media/how-to-use-event-grid/filtering-events.png)

1. A continuación, agregue un paso para consumir este evento y buscar el correo electrónico. Hay varias cuentas de correo electrónico que puede usar para recibir eventos. También puede configurar condiciones sobre cuándo enviar una alerta de correo electrónico.

    ![select-email-action](./media/how-to-use-event-grid/select-email-action.png)

1. Seleccione __Enviar un correo electrónico__ y rellene los parámetros. En el asunto, puede incluir el __tipo de evento__ y el __tema__ para ayudar a filtrar los eventos. También puede incluir un vínculo a la página del área de trabajo para las ejecuciones en el cuerpo del mensaje. 

    ![configure-email-body](./media/how-to-use-event-grid/configure-email-body.png)

1. Para guardar esta acción, seleccione **Guardar como** en la esquina izquierda de la página. En la barra de la derecha que aparece, confirme la creación de esta acción.

    ![confirm-logic-app-create](./media/how-to-use-event-grid/confirm-logic-app-create.png)


### <a name="example-data-drift-triggers-retraining"></a>Ejemplo: Reentrenamiento de desencadenador de desfase de datos

Los modelos quedan obsoletos a lo largo del tiempo y conservan la utilidad en el contexto en el que se ejecutan. Una manera de saber si es el momento de volver a entrenar el modelo consiste en detectar el desfase de los datos. 

En este ejemplo se muestra cómo usar Event Grid con una aplicación lógica de Azure para desencadenar el nuevo entrenamiento. El ejemplo desencadena una canalización de Azure Data Factory cuando se produce el desfase de datos entre los conjuntos de datos de entrenamiento y de servicio de un modelo.

Antes de empezar, realice las acciones siguientes:

* Configure un monitor de conjunto de datos para [detectar el desfase de datos]( https://aka.ms/datadrift) en un área de trabajo
* Cree una [canalización de Azure Data Factory](https://docs.microsoft.com/azure/data-factory/) publicada.

En este ejemplo, se usa una canalización de Data Factory simple para copiar archivos en un almacén de blobs y ejecutar una canalización de Machine Learning publicada. Para más información sobre este escenario, vea cómo configurar un [paso de Machine Learning en Azure Data Factory](https://docs.microsoft.com/azure/data-factory/transform-data-machine-learning-service)

![adf-mlpipeline-stage](./media/how-to-use-event-grid/adf-mlpipeline-stage.png)

1. Comience con la creación de la aplicación lógica. Vaya a [Azure Portal](https://portal.azure.com), busque Aplicaciones lógicas y seleccione Crear.

    ![search-logic-app](./media/how-to-use-event-grid/search-for-logic-app.png)

1. Rellene la información solicitada. Para simplificar la experiencia, use la misma suscripción y el mismo grupo de recursos de la canalización de Azure Data Factory y el área de trabajo de Azure Machine Learning.

    ![set-up-logic-app-for-adf](./media/how-to-use-event-grid/set-up-logic-app-for-adf.png)

1. Una vez que haya creado la aplicación lógica, seleccione __Cuando se produce un evento de recursos de Event Grid__. 

    ![select-event-grid-trigger](./media/how-to-use-event-grid/select-event-grid-trigger.png)

1. Inicie sesión y rellene los detalles del evento. Establezca el __nombre del recurso__ en el nombre del área de trabajo. Establezca el __tipo de evento__ en __DatasetDriftDetected__.

    ![login-and-add-event](./media/how-to-use-event-grid/login-and-add-event.png)

1. Agregue un nuevo paso y busque __Azure Data Factory__. Seleccione __Crear una ejecución de canalización__. 

    ![create-adfpipeline-run](./media/how-to-use-event-grid/create-adfpipeline-run.png)

1. Inicie sesión y especifique la canalización de Azure Data Factory publicada que se va a ejecutar.

    ![specify-adf-pipeline](./media/how-to-use-event-grid/specify-adf-pipeline.png)

1. Guarde y cree la aplicación lógica con el botón **Guardar** de la parte superior izquierda de la página. Para ver la aplicación, vaya al área de trabajo en [Azure Portal](https://portal.azure.com) y haga clic en **Eventos**.

    ![show-logic-app-webhook](./media/how-to-use-event-grid/show-logic-app-webhook.png)

Ahora la canalización de Data Factory se desencadena cuando se produce el desfase. Vea los detalles de la ejecución de desfase de datos y la canalización de aprendizaje automático en el [nuevo portal de áreas de trabajo](https://ml.azure.com). 

![view-in-workspace](./media/how-to-use-event-grid/view-in-workspace.png)

### <a name="example-deploy-a-model-based-on-tags"></a>Ejemplo: Implementación de un modelo basado en etiquetas

Un objeto de modelo de Azure Machine Learning contiene parámetros sobre los que puede dinamizar las implementaciones, como el nombre del modelo, la versión, la etiqueta y la propiedad. El evento de registro del modelo puede desencadenar un punto de conexión y puede usar una función de Azure para implementar un modelo en función del valor de esos parámetros.

Para obtener un ejemplo, vea el repositorio [https://github.com/Azure-Samples/MachineLearningSamples-NoCodeDeploymentTriggeredByEventGrid](https://github.com/Azure-Samples/MachineLearningSamples-NoCodeDeploymentTriggeredByEventGrid) y siga los pasos del archivo **Léame**.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre Event Grid y pruebe los eventos de Azure Machine Learning:

- [Una introducción a Azure Event Grid](../event-grid/overview.md)

- [Esquema de eventos para Azure Machine Learning](../event-grid/event-schema-machine-learning.md)

