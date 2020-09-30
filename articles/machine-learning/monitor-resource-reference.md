---
title: Referencia de datos de supervisión | Microsoft Docs
titleSuffix: Azure Machine Learning
description: Obtenga información sobre los datos y recursos recopilados para Azure Machine Learning y disponibles en Azure Monitor. Azure Monitor recopila y muestra datos sobre el área de trabajo de Azure Machine Learning y le permite ver las métricas, establecer alertas y analizar los datos registrados.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.reviewer: larryfr
ms.author: aashishb
author: aashishb
ms.date: 04/27/2020
ms.openlocfilehash: 405b0aa051d0d1142d7dd4ccbf2bca4ef9cc3545
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89650595"
---
# <a name="azure-machine-learning-monitoring-data-reference"></a>Referencia de datos de supervisión de Azure Machine Learning

Obtenga información sobre los datos y los recursos recopilados por Azure Monitor de su área de trabajo de Azure Machine Learning. Consulte [Supervisión de Azure Machine Learning](monitor-azure-machine-learning.md) para obtener más información sobre la recopilación y el análisis de datos de supervisión.

## <a name="resource-logs"></a>Registros del recurso

En la tabla siguiente se enumeran las propiedades de los registros de recursos de Azure Machine Learning cuando se recopilan en registros de Azure Monitor o Azure Storage.

### <a name="amlcomputejobevents-table"></a>Tabla AmlComputeJobEvents

| Propiedad | Descripción |
|:--- |:---|
| TimeGenerated | Hora a la que se generó la entrada de registro |
| OperationName | Nombre de la operación asociada con el evento de registro |
| Category | Nombre del evento de registro, AmlComputeClusterNodeEvent |
| JobId | Id. del trabajo enviado |
| ExperimentId | Id. del experimento |
| ExperimentName | Nombre del experimento |
| CustomerSubscriptionId | SubscriptionId adonde se envían el experimento y el trabajo |
| WorkspaceName | Nombre del área de trabajo de aprendizaje automático |
| ClusterName | Nombre del clúster |
| ProvisioningState | Estado del envío del trabajo |
| ResourceGroupName | Nombre del grupo de recursos |
| JobName | Nombre del trabajo |
| ClusterId | Id. del clúster |
| EventType | Tipo de evento de trabajo, p. ej., JobSubmitted, JobRunning, JobFailed, JobSucceeded, etc. |
| ExecutionState | Estado del trabajo (ejecución), p. ej., Queued, Running, Succeeded, Failed |
| ErrorDetails | Detalles del error de trabajo |
| CreationApiVersion | Versión de API que se usa para crear el trabajo |
| ClusterResourceGroupName | Nombre del grupo de recursos del clúster |
| TFWorkerCount | Recuento de los trabajos de TF |
| TFParameterServerCount | Recuento del servidor de parámetro TF |
| ToolType | Tipo de herramienta utilizada |
| RunInContainer | Marca que describe si el trabajo se debe ejecutar dentro de un contenedor |
| JobErrorMessage | mensaje detallado de error de trabajo |
| NodeId | Id. del nodo creado en el que se está ejecutando el trabajo |

### <a name="amlcomputeclusterevents-table"></a>Tabla AmlComputeClusterEvents

| Propiedad | Descripción |
|:--- |:--- |
| TimeGenerated | Hora a la que se generó la entrada de registro |
| OperationName | Nombre de la operación asociada con el evento de registro |
| Category | Nombre del evento de registro, AmlComputeClusterNodeEvent |
| ProvisioningState | Estado de aprovisionamiento del clúster |
| ClusterName | Nombre del clúster |
| ClusterType | Tipo de clúster |
| CreatedBy | Usuario que creó el clúster |
| CoreCount | Recuento de núcleos del clúster |
| VmSize | Tamaño de la máquina virtual del clúster |
| VmPriority | Prioridad de los nodos creados dentro de un clúster dedicado/LowPriority |
| ScalingType | Tipo de escalado del clúster manual/automático |
| InitialNodeCount | Número inicial de nodos del clúster |
| MinimumNodeCount | Número mínimo de nodos del clúster |
| MaximumNodeCount | Número máximo de nodos del clúster |
| NodeDeallocationOption | Cómo se debe desasignar el nodo |
| Publicador | Publicador del tipo de clúster |
| Oferta | Oferta con la que se crea el clúster |
| SKU | SKU del nodo o la máquina virtual creada dentro del clúster |
| Versión | Versión de la imagen usada mientras se crea el nodo o la máquina virtual |
| SubnetId | SubnetId del clúster |
| AllocationState | Estado de asignación del clúster |
| CurrentNodeCount | Número actual de nodos del clúster |
| TargetNodeCount | Recuento de nodos de destino del clúster durante el escalado o la reducción vertical |
| EventType | Tipo de evento durante la creación del clúster. |
| NodeIdleTimeSecondsBeforeScaleDown | Tiempo de inactividad en segundos antes de que el clúster se reduzca verticalmente |
| PreemptedNodeCount | Número de nodos con prioridad del clúster |
| IsResizeGrow | Marca que indica que el clúster se está escalando verticalmente |
| VmFamilyName | Nombre de la familia de la VM de los nodos que se pueden crear dentro del clúster |
| LeavingNodeCount | Número de nodos de salida del clúster |
| UnusableNodeCount | Número de nodos no utilizables del clúster |
| IdleNodeCount | Número de nodos inactivos del clúster |
| RunningNodeCount | Número de nodos en ejecución del clúster |
| PreparingNodeCount | Número de nodos en preparación del clúster |
| QuotaAllocated | Cuota asignada del clúster |
| QuotaUtilized | Cuota utilizada del clúster |
| AllocationStateTransitionTime | Tiempo de transición de un estado a otro |
| ClusterErrorCodes | Código de error recibido durante la creación o el escalado de un clúster |
| CreationApiVersion | Versión de API usada durante la creación del clúster |

### <a name="amlcomputeclusternodeevents-table"></a>Tabla AmlComputeClusterNodeEvents

| Propiedad | Descripción |
|:--- |:--- |
| TimeGenerated | Hora a la que se generó la entrada de registro |
| OperationName | Nombre de la operación asociada con el evento de registro |
| Category | Nombre del evento de registro, AmlComputeClusterNodeEvent |
| ClusterName | Nombre del clúster |
| NodeId | Id. del nodo del clúster creado |
| VmSize | El tamaño de la máquina virtual del nodo |
| VmFamilyName | La familia de la máquina virtual a la que pertenece el nodo |
| VmPriority | Prioridad del nodo creado Dedicado/LowPriority |
| Publicador | Publicar de la imagen de la máquina virtual, por ejemplo, microsoft-dsvm |
| Oferta | Oferta asociada con la creación de la VM |
| SKU | SKU del nodo o la máquina virtual creada |
| Versión | Versión de la imagen usada mientras se crea el nodo o la máquina virtual |
| ClusterCreationTime | Hora en la que fue creado el clúster |
| ResizeStartTime | Hora en que comenzó el escalado o la reducción vertical del clúster |
| ResizeEndTime | Hora en que terminó el escalado o la reducción vertical del clúster |
| NodeAllocationTime | Hora a la que se asignó el nodo |
| NodeBootTime | Hora a la que se arrancó el nodo |
| StartTaskStartTime | Hora a la que se asignó la tarea a un nodo y se inició |
| StartTaskEndTime | Hora a la que se asignó la tarea a un nodo y se terminó |
| TotalE2ETimeInSeconds | Tiempo total en que el nodo estuvo activo |

### <a name="metrics"></a>Métricas

En las tablas siguientes se enumeran las métricas de plataforma recopiladas para Azure Machine Learning. Todas las métricas se almacenan en el espacio de nombres **Área de trabajo de Azure Machine Learning**.

**Model**

| Métrica | Unidad | Descripción |
| ----- | ----- | ----- |
| Error al realizar la implementación del modelo | Count | Número de implementaciones de modelos que dieron error. |
| Implementación del modelo iniciada | Count | Número de implementaciones de modelos que se iniciaron. |
| Implementación de modelo correcta | Count | El número de implementaciones de modelos que se realizaron correctamente. |
| Error en el registro de modelo | Count | El número de registros de modelos que dieron error. |
| Registro de modelo realizado correctamente | Count | El número de registros de modelo que se realizaron correctamente. |

**Cuota**

La información de cuota es solo para proceso de Azure Machine Learning.

| Métrica | Unidad | Descripción |
| ----- | ----- | ----- |
| Núcleos activos | Count | El número de núcleos de proceso activos. |
| Nodos activos | Count | El número de nodos de proceso activos. |
| Núcleos inactivos | Count | El número de núcleos de proceso inactivos. |
| Nodos inactivos | Count | El número de nodos de ejecución inactivos. |
| Núcleos de salida | Count | Número de núcleos de salida. |
| Nodos de salida | Count | Número de nodos de salida. |
| Núcleos con prioridad | Count | Número de núcleos con prioridad. |
| Nodos con prioridad | Count | Número de nodos con prioridad. |
| Porcentaje de uso de la cuota | Percent | Porcentaje de cuota usada. |
| Núcleos totales | Count | Los núcleos totales. |
| Nodos totales | Count | Los núcleos totales. |
| Núcleos no utilizables | Count | El número de núcleos no utilizables. |
| Nodos no utilizables | Count | El número de nodos no utilizables. |

A continuación se indican las dimensiones que se pueden usar para filtrar las métricas de cuota:

| Dimensión | Métrica(s) disponible(s) con | Descripción |
| ---- | ---- | ---- |
| Cluster Name | Todas las métricas de cuota | Nombre de la instancia de proceso. |
| Nombre de familia de VM | Porcentaje de uso de la cuota | El nombre de la familia de máquinas virtuales que usa el clúster. |
| Prioridad de VM | Porcentaje de uso de la cuota | La prioridad de la VM.

**Recurso**

| Métrica | Unidad | Descripción |
| ----- | ----- | ----- |
| CpuUtilization | Percent | Porcentaje de la CPU utilizado para un nodo concreto durante una ejecución o trabajo. Esta métrica solo se publica cuando se ejecuta un trabajo en un nodo. Un trabajo puede usar uno o varios nodos. Esta métrica se publica por nodo. |
| GpuUtilization | Percent | Porcentaje de la GPU utilizado para un nodo concreto durante una ejecución o trabajo. Un nodo puede tener una o varias GPU. Esta métrica se publica por GPU por nodo. |

A continuación se indican las dimensiones que se pueden usar para filtrar las métricas de recursos:

| Dimensión | Descripción |
| ----- | ----- |
| CreatedTime | |
| deviceId | Identificador del dispositivo (GPU). Solo está disponible para GpuUtilization. |
| NodeId | Identificador del nodo creado en el que se está ejecutando el trabajo |
| RunId | Identificador de la ejecución o el trabajo. |

**Ejecutar**

Información sobre las ejecuciones de aprendizaje.

| Métrica | Unidad | Descripción |
| ----- | ----- | ----- |
| Ejecuciones finalizadas | Count | Número de ejecuciones completadas. |
| Ejecuciones con error | Count | Número de ejecuciones con error. |
| Ejecuciones iniciadas | Count | Número de ejecuciones iniciadas. |

A continuación se indican las dimensiones que se pueden usar para filtrar las métricas de ejecución:

| Dimensión | Descripción |
| ---- | ---- |
| ComputeType | El tipo de proceso que usó la ejecución. |
| PipelineStepType | El tipo de [PipelineStep](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinestep?view=azure-ml-py&preserve-view=true) que se usó en la ejecución. |
| PublishedPipelineId | El Id. de la canalización publicada utilizada en la ejecución. |
| RunType | El tipo de ejecución. |

Los valores válidos para la dimensión RunType son:

| Value | Descripción |
| ----- | ----- |
| Experimento | Ejecuciones que no son de canalización. |
| PipelineRun | Una ejecución de canalización, que es el elemento primario de una StepRun. |
| StepRun | Una ejecución para un paso de canalización. |
| ReusedStepRun | Una ejecución de un paso de canalización que reutiliza una ejecución anterior. |

## <a name="see-also"></a>Consulte también

- Consulte [Supervisión de Azure Machine Learning](monitor-azure-machine-learning.md) para obtener una descripción de la supervisión de Azure Machine Learning.
- Para más información sobre la supervisión de recursos de Azure, consulte [Supervisión de recursos de Azure con Azure Monitor](/azure/azure-monitor/insights/monitor-azure-resource).
