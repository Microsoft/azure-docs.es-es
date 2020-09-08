---
title: Ediciones Enterprise y Basic de Azure Machine Learning
description: Conozca las diferencias entre las ediciones de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
author: j-martens
ms.author: jmartens
ms.date: 06/11/2020
ms.openlocfilehash: fbe3f0109ca3af2cf9ff44061c7882e6bd72c8a6
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88508609"
---
# <a name="enterprise-preview-and-basic-editions-of-azure-machine-learning"></a>Ediciones Enterprise (versión preliminar) y Basic de Azure Machine Learning 

Azure Machine Learning ofrece dos ediciones adaptadas a sus necesidades de aprendizaje automático. Estas ediciones determinan qué herramientas de aprendizaje automático están disponibles para los desarrolladores y científicos de datos en su área de trabajo.

## <a name="choose-an-edition"></a>Elección de una edición

La edición se asigna cada vez que se crea un área de trabajo. Los clientes son responsables de los costos en los que se incurre en el proceso y en otros recursos de Azure durante este tiempo. Descubra cómo [administrar los costos de Azure Machine Learning](concept-plan-manage-cost.md).

Aprenda a [actualizar un área de trabajo de la edición Basic a Enterprise (versión preliminar)](how-to-manage-workspace.md#upgrade). 

## <a name="whats-in-each-edition"></a>Qué hay en cada edición

### <a name="data-for-machine-learning-capabilities"></a>Capacidades de datos para Machine Learning  

| Capacidades                     | Edición                 |
|------------------------------------------------------------------------------------|:-----------:|
| Etiquetado: [Creación y administración de proyectos de etiquetado](tutorial-labeling.md) en Studio (web)                                                | All                     |
| Etiquetado: Etiquetador en Studio (web)                                    | All                     |
| Etiquetado: Uso de personal móvil                               | All                     |
| Etiquetado: [Clasificación de imágenes y detección de objetos asistidas por ML](how-to-label-images.md)                  | Solo edición Enterprise |
| Conjuntos de datos y almacenes de datos: creación y administración en Python                       | All                     |
| Conjuntos de datos y almacenes de datos: creación y administración en Studio (web)                         | All                     |
| Desfase: Visualización y administración de monitores de conjuntos de datos en Python                           | All                     |
| Desfase: Visualización y administración de monitores de conjuntos de datos en Studio (web)                            | Solo edición Enterprise |


<br/>
<br/>

### <a name="automated-training-capabilities-automl"></a>Capacidades de aprendizaje automatizado (AutoML)

| Capacidades    | Edición                 |
|------------------------------------------------------------------------------------|:-----------:|
| Creación y ejecución de [experimentos de AutoML en cuadernos](how-to-configure-auto-train.md)               | All                     |
| Creación y ejecución de [experimentos de AutoML en Studio (web)](how-to-use-automated-ml-for-ml-models.md)   | Solo edición Enterprise |
| Capacidades de previsión de AutoML líderes del sector             | Solo edición Enterprise |
| Compatibilidad con aprendizaje profundo y otros aprendizajes avanzados | Solo edición Enterprise |
| Tareas de regresión y clasificación de compatibilidad con datos de gran tamaño (hasta 100 GB)                     | Solo edición Enterprise |


<br/>
<br/>

### <a name="responsible-machine-learning"></a>Machine Learning responsable

| Capacidades    | Edición                 |
|------------------------------------------------------------------------------------|:-----------:|
| [Explicación del modelo](how-to-machine-learning-interpretability-automl.md)                                              | All                     |
| [Privacidad diferencial](how-to-differential-privacy.md)                          | All                     |
| Etiquetas personalizadas para implementar hojas de datos    | All                     |
| Integración con AzureML de equidad                                      | All                     |

<br/>
<br/>


### <a name="build-and-train-capabilities"></a>Capacidades de compilación y entrenamiento

| Capacidades    | Edición                 |
|------------------------------------------------------------------------------------|:-----------:|
| Integración con Visual Studio Code                                                     | All                     |
| Aprendizaje de refuerzo                                                             | All                     |
| Interfaz de usuario de experimentación                                                                 | All                     |
| Jupyter, integración con JupyterLab                                                    | All                     |
| Compatibilidad con el SDK de Python                                                                 | All                     |
| Compatibilidad con el SDK de R                                                                      | All                     |
| Canalizaciones de ML: Creación, ejecución y publicación en Python                           | All                     |
| Canalizaciones de ML: Creación, edición y eliminación de ejecuciones programadas de canalizaciones en Python| All                     |
| Canalizaciones de ML: Creación de puntos de conexión de canalización en el SDK de Python                                   | All                     |
| Canalizaciones de ML: Visualización de los detalles de la ejecución en Studio (Web)                                              | All                     |
| Canalizaciones de ML: Creación, ejecución, visualización y publicación en el diseñador                  | Solo edición Enterprise |
| Canalizaciones de ML: Creación de puntos de conexión de canalización en el diseñador | Solo edición Enterprise |
| Instancias de proceso administradas para cuadernos integrados                                 | All                     |


<br/>
<br/>

### <a name="deployment-and-model-management-capabilities"></a>Capacidades de implementación y administración de modelos

| Capacidades                            | Edición                 |
|------------------------------------------------------------------------------------|:-----------:|
| La extensión Azure DevOps para Machine Learning y la CLI de Azure ML                 | All                     |
| [Integración con Event Grid](how-to-use-event-grid.md)                                                             | All                     |
| Integración de Azure Stream Analytics con Azure Machine Learning                       | All                     |
| Creación de canalizaciones de Machine Learning en el SDK                                                         | All                     |
| Inferencia por lotes                                                                  | All                     |
| Modelos acelerados mediante hardware basados en FPGA                                             | All                     |
| Generación de perfiles de modelos                                                                    | All                     |

<br/>
<br/>

### <a name="security-governance-and-control-capabilities"></a>Capacidades de seguridad, gobernanza y control

| Capacidades     | Edición                 |
|------------------------------------------------------------------------------------|:-----------:|
| Compatibilidad con el [control de acceso basado en rol de Azure (RBAC de Azure)](how-to-assign-roles.md)                                           | All                     |
| Compatibilidad del proceso con [redes virtuales (VNet)](how-to-enable-virtual-network.md)                                         | All                     |
| Autenticación de puntos de conexión de puntuación                                                    | All                     |
| [Private Link del área de trabajo](how-to-configure-private-link.md)                                                            | All                     |
| [Administración de cuotas](how-to-manage-quotas.md) entre áreas de trabajo                                                 | Solo edición Enterprise |

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre lo que está disponible en la [página de precios e información general de las ediciones](https://azure.microsoft.com/pricing/details/machine-learning/) de Azure Machine Learning. 

Aprenda a [actualizar un área de trabajo de la edición Basic a Enterprise](how-to-manage-workspace.md#upgrade). 
