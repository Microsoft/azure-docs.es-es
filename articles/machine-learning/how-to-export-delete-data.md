---
title: Exportación o eliminación de los datos del área de trabajo
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo exportar o eliminar el área de trabajo con Azure Machine Learning Studio, la CLI, el SDK y las API REST autenticadas.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: jmartens
author: lobrien
ms.author: laobri
ms.date: 04/24/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: fd3abdfd9b0bc0e658caa1cc5ab5c5a7edfda453
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494041"
---
# <a name="export-or-delete-your-machine-learning-service-workspace-data"></a>Exportación o eliminación de los datos del área de trabajo de Machine Learning Service

[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En Azure Machine Learning, puede exportar o eliminar los datos del área de trabajo mediante la interfaz gráfica del portal o el SDK de Python. En este artículo se describen ambas opciones.

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-dsr-and-stp-note.md)]

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

## <a name="control-your-workspace-data"></a>Control de los datos del área de trabajo

Los datos en el producto que almacena Azure Machine Learning están disponibles para su exportación y eliminación. Puede exportarlos y eliminarlos mediante Azure Machine Learning Studio, la CLI y el SDK. Se puede acceder a los datos de telemetría mediante el portal de privacidad de Azure. 

En Azure Machine Learning, los datos personales se componen de información de usuario procedente de los documentos del historial de ejecuciones. 

## <a name="delete-high-level-resources-using-the-portal"></a>Eliminación de recursos de alto nivel mediante el portal

Cuando se crea un área de trabajo, Azure crea una serie de recursos en el grupo de recursos:

- La propia área de trabajo
- Una cuenta de almacenamiento
- Un registro de contenedor
- Una instancia de Application Insights
- Un almacén de claves

Estos recursos se pueden eliminar seleccionándolos en la lista y eligiendo **Eliminar**. 

:::image type="content" source="media/how-to-export-delete-data/delete-resource-group-resources.png" alt-text="Captura de pantalla del portal, con el icono Eliminar resaltado":::

Los documentos del historial de ejecución, que pueden contener información personal del usuario, se almacenan en la cuenta del almacenamiento de blobs, en las subcarpetas de `/azureml`. Puede descargar y eliminar los datos del portal.

:::image type="content" source="media/how-to-export-delete-data/storage-account-folders.png" alt-text="Captura de pantalla del directorio azureml en la cuenta de almacenamiento, en el portal":::

## <a name="export-and-delete-machine-learning-resources-using-azure-machine-learning-studio"></a>Exportación y eliminación de los recursos de aprendizaje automático con Azure Machine Learning Studio

Azure Machine Learning Studio proporciona una vista unificada de los recursos de aprendizaje automático, como cuadernos, conjuntos de información, modelos y experimentos. Azure Machine Learning Studio enfatiza la conservación de un registro de datos y experimentos. Los recursos de cálculo, como las canalizaciones y los recursos de procesos, se pueden eliminar mediante el explorador. Para eliminar esos recursos, vaya al recurso en cuestión y seleccione **Eliminar**. 

Los conjuntos de datos se pueden anular del registro y los experimentos se pueden archivar, pero tenga en cuenta que estas operaciones no eliminan los datos. Para quitar completamente los datos, los conjuntos de datos y los datos de ejecución, estos deben eliminarse en el nivel de almacenamiento. La eliminación en el nivel de almacenamiento se realiza mediante el portal, tal como se ha descrito anteriormente.

Puede descargar los artefactos de aprendizaje de las ejecuciones experimentales mediante Studio. Elija el **experimento** y **ejecútelo**. Seleccione **Salida y registros** y elija los artefactos específicos que quiera descargar. Seleccione **...** y **Descargar**.

Puede descargar un modelo registrado seleccionando el **modelo** que quiera y haciendo clic en **Descargar**. 

:::image type="contents" source="media/how-to-export-delete-data/model-download.png" alt-text="Captura de pantalla de la página del modelo de Studio con la opción de descarga resaltada":::

## <a name="export-and-delete-resources-using-the-python-sdk"></a>Exportación y eliminación de los recursos con el SDK de Python

Puede descargar los resultados de una ejecución determinada mediante: 

```python
# Retrieved from Azure Machine Learning web UI
run_id = 'aaaaaaaa-bbbb-cccc-dddd-0123456789AB'
experiment = ws.experiments['my-experiment']
run = next(run for run in ex.get_runs() if run.id == run_id)
metrics_output_port = run.get_pipeline_output('metrics_output')
model_output_port = run.get_pipeline_output('model_output')

metrics_output_port.download('.', show_progress=True)
model_output_port.download('.', show_progress=True)
```

Los siguientes recursos de aprendizaje automático se pueden eliminar mediante el SDK de Python: 

| Tipo | Llamada a función | Notas | 
| --- | --- | --- |
| `Workspace` | [`delete`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py#delete-delete-dependent-resources-false--no-wait-false-) | Use `delete-dependent-resources` para poner en cascada el proceso de eliminación. |
| `Model` | [`delete`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model(class)?view=azure-ml-py#delete--) | | 
| `ComputeTarget` | [`delete`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.computetarget?view=azure-ml-py#delete--) | |
| `WebService` | [`delete`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice%28class%29?view=azure-ml-py) | | 

