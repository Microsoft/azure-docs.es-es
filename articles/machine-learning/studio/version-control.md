---
title: 'ML Studio (clásico): Administración del ciclo de vida de las aplicaciones: Azure'
description: Aplicación de procedimientos recomendados de administración del ciclo de vida de las aplicaciones en Azure Machine Learning Studio (clásico)
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.date: 10/27/2016
ms.openlocfilehash: 1f577c43b6602a78dd585d1b998110c708aa54b0
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432106"
---
# <a name="application-lifecycle-management-in-azure-machine-learning-studio-classic"></a>Administración del ciclo de vida de las aplicaciones en Azure Machine Learning Studio (clásico)

**SE APLICA A:**  ![yes](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


Azure Machine Learning Studio (clásico) es una herramienta para desarrollar experimentos de aprendizaje automático que se aplican en la plataforma de nube de Azure. Es como IDE de Visual Studio y servicio en la nube escalable combinados en una sola plataforma. Es posible incorporar a Azure Machine Learning Studio (clásico) procedimientos estándar de administración del ciclo de vida de las aplicaciones (ALM), desde el control de versiones de diversos recursos a la ejecución e implementación automatizadas. En este artículo se analizan algunas de las opciones y enfoques.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="versioning-experiment"></a>Control de versiones del experimento
Se recomiendan dos formas de aplicar el control de versiones a los experimentos. Puede confiar en el historial de ejecución integrado o exportar el experimento en formato JSON y administrarlo de forma externa. Cada enfoque tiene sus ventajas e inconvenientes.

### <a name="experiment-snapshots-using-run-history"></a>Instantáneas del experimento mediante el historial de ejecución
En el modelo de ejecución del experimento de aprendizaje de Azure Machine Learning Studio (clásico), se envía una instantánea inmutable del experimento al programador de trabajos cada vez que hace clic en el botón **Ejecutar** del editor de experimentos. Para ver esta lista de instantáneas, haga clic en **Historial de ejecución** en la barra de comandos de la vista del editor de experimentos.

![Botón Historial de ejecución](./media/version-control/runhistory.png)

A continuación, puede abrir la instantánea en modo de bloqueo haciendo clic en el nombre del experimento en el momento en que este se envió para su ejecución y se tomó la instantánea. Tenga en cuenta que solo el primer elemento de la lista, que representa el experimento actual, está en estado editable. Tenga en cuenta además que cada instantánea puede tener también diversos estados, incluidos completado (ejecución parcial), de error, de error (ejecución parcial) o de borrador.

![Lista del historial de ejecuciones](./media/version-control/runhistorylist.png)

Una vez que se abre, puede guardar el experimento de la instantánea como experimento nuevo para luego modificarlo. Si la instantánea del experimento contiene recursos como modelos entrenados, transformaciones o conjuntos de datos que tienen versiones actualizadas, la instantánea conserva las referencias a la versión original cuando se tomó. Si guarda la instantánea bloqueada como un experimento nuevo, Azure Machine Learning Studio (clásico) detecta la existencia de una versión más reciente de estos recursos y los actualiza automáticamente en el experimento nuevo.

Si elimina el experimento, se eliminan todas las instantáneas de dicho experimento.

### <a name="exportimport-experiment-in-json-format"></a>Exportar o importar experimento en formato JSON
Las instantáneas del historial de ejecución conservan una versión inmutable del experimento en Azure Machine Learning Studio (clásico) cada vez que se envía para su ejecución. También puede guardar una copia local del experimento y protegerla en el sistema de control de código fuente de su preferencia, como Team Foundation Server, para luego volver a crear un experimento a partir de ese archivo local. Puede usar los commandlets [Azure Machine Learning PowerShell](https://aka.ms/amlps)[*Export-AmlExperimentGraph*](https://github.com/hning86/azuremlps#export-amlexperimentgraph) e [*Import-AmlExperimentGraph*](https://github.com/hning86/azuremlps#import-amlexperimentgraph) para hacerlo.

El archivo JSON es una representación textual del gráfico del experimento, en la que podría incluirse una referencia a los recursos del área de trabajo, como un conjunto de datos o un modelo entrenado. No contiene una versión serializada del recurso. Si intenta importar el documento JSON de nuevo al área de trabajo, esos recursos a los que se hace referencia ya deben existir con los mismos identificadores de recurso a los que se hace referencia en el experimento. En caso contrario, no podrá tener acceso al experimento importado.

## <a name="versioning-trained-model"></a>Control de versiones de un modelo entrenado
Un modelo entrenado en Azure Machine Learning Studio (clásico) se serializa en un formato conocido como un archivo iLearner (`.iLearner`) y se almacena en la cuenta de Azure Blob Storage asociada al área de trabajo. Una forma de obtener una copia del archivo iLearner es a través de la API de reentrenamiento. En [este artículo](/azure/machine-learning/studio/retrain-machine-learning-model) se explica cómo funciona la API de reentrenamiento. Pasos de alto nivel:

1. Actualice el experimento de entrenamiento.
2. Agregue un puerto de salida de servicio web al módulo Entrenar modelo o al módulo que produce el modelo entrenado, como el módulo Optimizar el modelo Hiperparámetros o Crear modelo R.
3. Ejecute el experimento de entrenamiento y, a continuación, impleméntelo como servicio web de entrenamiento de modelos.
4. Llame al punto de conexión BES del servicio web de entrenamiento y especifique el nombre de archivo iLearner deseado y la ubicación de la cuenta de Blob Storage donde se almacenará.
5. Recopile el archivo iLearner producido una vez finalizada la llamada a BES.

Otra forma de recuperar el archivo iLearner es a través del commandlet [*Download-AmlExperimentNodeOutput*](https://github.com/hning86/azuremlps#download-amlexperimentnodeoutput) de PowerShell. Esto podría ser más fácil si solo desea obtener una copia del archivo .iLearner sin necesidad de reciclar el modelo de manera programática.

Una vez que tiene el archivo iLearner que contiene el modelo entrenado, puede emplear su propia estrategia de control de versiones. La estrategia puede ser tan simple como aplicar un prefijo o postfijo como una convención de nomenclatura y solo dejar el archivo iLearner en Blob Storage, o bien copiarlo e importarlo en el sistema de control de versiones.

El archivo iLearner guardado se puede usar para puntuar a través de servicios web implementados.

## <a name="versioning-web-service"></a>Control de versiones de un servicio web
Puede implementar dos tipos de servicios web a partir de un experimento de Azure Machine Learning Studio (clásico). El servicio web clásico está estrechamente vinculado con el experimento, así como con el área de trabajo. El nuevo servicio web usa el marco de Azure Resource Manager y ya no está vinculado con el área de trabajo ni el experimento original.

### <a name="classic-web-service"></a>Servicio web clásico
Para aplicar control de versiones a un servicio web clásico, puede aprovechar la construcción del punto de conexión de servicio web. Este es un flujo típico:

1. A partir de su experimento predictivo, implementa un nuevo servicio web clásico, que contiene un punto de conexión predeterminado.
2. Crea un nuevo punto de conexión llamado ep2, que muestra la versión actual del experimento o modelo entrenado.
3. Vuelve y actualiza el experimento predictivo y el modelo entrenado.
4. Vuelve a implementar el experimento predictivo, que actualizará entonces el punto de conexión predeterminado. Sin embargo, esto no modificará ep2.
5. Crea un punto de conexión adicional llamado ep3, que muestra la nueva versión del experimento y modelo entrenado.
6. Vuelva al paso 3 si es necesario.

Con el tiempo, es posible que se creen muchos puntos de conexión en el mismo servicio web. Cada uno de ellos representa una copia a un momento dado del experimento que contiene la versión a un momento dado del modelo entrenado. Podrás usar entonces lógica externa para determinar el punto de conexión al que llamar, lo que efectivamente significa seleccionar una versión del modelo entrenado para la ejecución de la puntuación.

También puede crear muchos puntos de conexión de servicio web idénticos y, a continuación, aplicar revisiones de diversas versiones del archivo iLearner al punto de conexión para lograr un efecto similar. En [este artículo](create-models-and-endpoints-with-powershell.md) se explica con más detalla cómo hacerlo.

### <a name="new-web-service"></a>Servicio web nuevo
Si crea un nuevo servicio web basado en Azure Resource Manager, la construcción del punto de conexión dejará de estar disponible. En su lugar, puede generar archivos de definición del servicio web (WSD), en formato JSON, a partir del experimento predictivo mediante el commandlet [Export-AmlWebServiceDefinitionFromExperiment](https://github.com/hning86/azuremlps#export-amlwebservicedefinitionfromexperiment) de PowerShell, o bien mediante el commandlet [*Export-AzMlWebservice*](https://docs.microsoft.com/powershell/module/az.machinelearning/export-azmlwebservice) de PowerShell a partir de un servicio web basado en Resource Manager implementado.

Una vez que tiene el archivo WSD exportado y controla sus versiones, también puede implementar WSD como servicio web nuevo en un plan de servicio web distinto en una región de Azure diferente. Basta con que se asegure de proporcionar la configuración adecuada de la cuenta de almacenamiento, así como el identificador del nuevo plan de servicio web. Para aplicar revisiones en diversos archivos iLearner, puede modificar el archivo WSD y actualizar la referencia de ubicación del modelo entrenado, además de implementarlo como nuevo servicio web.

## <a name="automate-experiment-execution-and-deployment"></a>Automatizar la ejecución e implementación del experimento
Un aspecto importante de ALM es poder automatizar el proceso de ejecución e implementación de la aplicación. En Azure Machine Learning Studio (clásico), puede hacerlo mediante el [módulo de PowerShell](https://aka.ms/amlps). Este es un ejemplo de los pasos completos que son pertinentes para un proceso de ejecución e implementación automatizadas de ALM estándar mediante el [módulo Azure Machine Learning Studio (classic) PowerShell (Azure Machine Learning Studio [clásico] PowerShell)](https://aka.ms/amlps). Cada paso está vinculado a uno o varios commandlets de PowerShell que puede usar para realizar ese paso.

1. [Cargue un conjunto de datos](https://github.com/hning86/azuremlps#upload-amldataset).
2. Copie un experimento de entrenamiento en el área de trabajo a partir de un [área de trabajo](https://github.com/hning86/azuremlps#copy-amlexperiment) o de [Galería](https://github.com/hning86/azuremlps#copy-amlexperimentfromgallery), o [importe](https://github.com/hning86/azuremlps#import-amlexperimentgraph) un experimento [exportado](https://github.com/hning86/azuremlps#export-amlexperimentgraph) desde el disco local.
3. [Actualice el conjunto de datos](https://github.com/hning86/azuremlps#update-amlexperimentuserasset) en el experimento de entrenamiento.
4. [Ejecute el experimento de entrenamiento](https://github.com/hning86/azuremlps#start-amlexperiment).
5. [Promocione el modelo entrenado](https://github.com/hning86/azuremlps#promote-amltrainedmodel).
6. [Copie un experimento predictivo](https://github.com/hning86/azuremlps#copy-amlexperiment) en el área de trabajo.
7. [Actualice el modelo entrenado](https://github.com/hning86/azuremlps#update-amlexperimentuserasset) en el experimento predictivo.
8. [Ejecute el experimento predictivo](https://github.com/hning86/azuremlps#start-amlexperiment).
9. [Implemente un servicio web](https://github.com/hning86/azuremlps#new-amlwebservice) a partir del experimento predictivo.
10. Pruebe el punto de conexión [RRS](https://github.com/hning86/azuremlps#invoke-amlwebservicerrsendpoint) o [BES](https://github.com/hning86/azuremlps#invoke-amlwebservicebesendpoint) de servicio web.

## <a name="next-steps"></a>Pasos siguientes
* Descargue el módulo [Azure Machine Learning Studio (classic) PowerShell](https://aka.ms/amlps) (Azure Machine Learning Studio [clásico] PowerShell) y empiece a automatizar sus tareas de ALM.
* Obtenga información sobre cómo [crear y administrar un gran número de modelos de ML con solo un único experimento](create-models-and-endpoints-with-powershell.md) a través de PowerShell y la API de reentrenamiento.
* Obtenga más información sobre cómo [implementar los servicios web de Azure Machine Learning](deploy-a-machine-learning-web-service.md).
