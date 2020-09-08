---
title: Acerca de los entornos de Azure Machine Learning
titleSuffix: Azure Machine Learning
description: En este artículo, obtendrá información sobre las ventajas de los entornos de aprendizaje automático, que permiten definiciones de dependencias de aprendizaje automático reproducibles, auditables y portátiles en distintos destinos de proceso.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: larryfr
author: BlackMist
ms.date: 07/08/2020
ms.openlocfilehash: cc4c39cf26f3ab8d1037222f967789bfbeca05ba
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88166780"
---
# <a name="what-are-azure-machine-learning-environments"></a>¿Qué son los entornos de Azure Machine Learning?
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

Los entornos de Azure Machine Learning son una encapsulación del entorno en el que se produce el entrenamiento de aprendizaje automático. Especifican los paquetes, las variables de entorno y la configuración de software de Python en torno a los scripts de entrenamiento y puntuación. También especifican los entornos de ejecución (Python, Spark o Docker). Los entornos son entidades administradas y con control de versiones en el área de trabajo de Machine Learning que permiten flujos de trabajo de aprendizaje automático reproducibles, auditables y portátiles en distintos destinos de proceso.

Puede usar un objeto de `Environment` en el proceso local para:
* Desarrollar el script de aprendizaje.
* Reutilizar el mismo entorno en el Proceso de Azure Machine Learning para el entrenamiento del modelo a escala.
* Implementar el modelo con ese mismo entorno.
* Vuelva a revisar el entorno en el que se entrenó un modelo existente.

En el diagrama siguiente se ilustra cómo puede usar un solo objeto `Environment` en la configuración de ejecución (para el entrenamiento) y en la configuración de inferencia e implementación (para las implementaciones de servicios web).

![Diagrama del entorno del flujo de trabajo de aprendizaje automático](./media/concept-environments/ml-environment.png)

El entorno, el destino de proceso y el script de entrenamiento constituyen todos juntos la configuración de ejecución: la especificación completa de una ejecución de entrenamiento.

## <a name="types-of-environments"></a>Tipos de entornos

Los entornos se pueden dividir a grandes rasgos en tres categorías: *mantenidos*, *administrados por el usuario* y *administrados por el sistema*.

Los entornos mantenidos los proporciona Azure Machine Learning y están disponibles en el área de trabajo de forma predeterminada. Están concebidos para usarse tal cual; contienen colecciones de paquetes y configuraciones de Python que le ayudarán a empezar a usar diferentes marcos de aprendizaje automático. Estos entornos creados previamente también permiten un tiempo de implementación más rápido. Para obtener una lista completa, consulte el [artículo sobre los entornos mantenidos](resource-curated-environments.md).

Para los entornos administrados por usuarios, es responsable de configurar el entorno e instalar todos los paquetes que necesita el script de entrenamiento en el destino de proceso. Conda no comprobará el entorno ni instalará nada por usted. Si va a definir un entorno propio, tendrá que enumerar `azureml-defaults` con la versión `>= 1.0.45` como dependencia pip. Este paquete contiene la funcionalidad necesaria para hospedar el modelo como un servicio web.

Use entornos administrados por el sistema cuando quiera que [Conda](https://conda.io/docs/) administre el entorno de Python y las dependencias de script. Se crea un entorno de Conda según el objeto de dependencias de Conda. Azure Machine Learning Service da por supuesto el uso de este tipo de entorno de forma predeterminada, debido a su utilidad en destinos de proceso remotos que no se pueden configurar manualmente.

## <a name="create-and-manage-environments"></a>Creación y administración de entornos

Puede crear entornos mediante los procesos siguientes:

* La definición de objetos `Environment` nuevos, ya sea mediante un entorno mantenido o la definición de dependencias propias.
* El uso de objetos `Environment` existentes desde el área de trabajo. Este enfoque permite la coherencia y la posibilidad de reproducir las dependencias.
* Importación desde una definición de entorno de Anaconda existente.
* Uso ed la CLI de Azure Machine Learning
* [Uso de la extensión de VS Code](how-to-manage-resources-vscode.md#create-environment)

Para ver ejemplos de código específicos, consulte la sección "Creación de un entorno" de [Uso de los entornos](how-to-use-environments.md#create-an-environment). Los entornos también se administran fácilmente a través del área de trabajo. Estos incluyen la funcionalidad siguiente:

* Los entornos se registran de forma automática en el área de trabajo cuando se envía un experimento. También se pueden registrar manualmente.
* Capture entornos del área de trabajo y úselos para el entrenamiento, la implementación o la modificación de la definición del entorno.
* El control de versiones permite ver los cambios en los entornos en el tiempo y garantiza la reproducibilidad.
* Puede compilar imágenes de Docker de forma automática desde los entornos.

Para ver ejemplos de código, consulte la sección "Administración de entornos" de [Uso de entornos](how-to-use-environments.md#manage-environments).

## <a name="environment-building-caching-and-reuse"></a>Compilación, almacenamiento en caché y reutilización de entornos

Azure Machine Learning Service compila definiciones de entorno en imágenes de Docker y entornos de Conda. También almacena en caché los entornos para que se puedan volver a usar en las ejecuciones de entrenamiento e implementaciones de punto de conexión de servicio posteriores. Para ejecutar un script de entrenamiento de forma remota, es necesario crear una imagen de Docker, mientras que una ejecución local puede usar un entorno de Conda directamente. 

### <a name="submitting-a-run-using-an-environment"></a>Envío de una ejecución mediante un entorno

La primera vez que se envía una ejecución remota mediante un entorno, Azure Machine Learning Service invoca una [tarea de compilación de ACR](https://docs.microsoft.com/azure/container-registry/container-registry-tasks-overview) en la instancia de Azure Container Registry (ACR) asociada al área de trabajo. Después, la imagen de Docker compilada se almacena en caché en la instancia de ACR del área de trabajo. Los entornos mantenidos están respaldados por imágenes de Docker que se almacenan en caché en la instancia de ACR global. Al inicio de la ejecución, el destino de proceso recupera la imagen de la instancia de ACR pertinente.

En el caso de ejecuciones locales, se crea un entorno de Docker o Conda en función de la definición del entorno. Luego, los scripts se ejecutan en el proceso de destino: un entorno de tiempo de ejecución local o un motor de Docker local.

### <a name="building-environments-as-docker-images"></a>Compilación de entornos como imágenes de Docker

Si la definición del entorno todavía no existe en la instancia de ACR del área de trabajo, se creará una imagen. La compilación de la imagen consta de dos pasos:

 1. Descarga de una imagen base y ejecución de todos los pasos de Docker;
 2. Creación de un entorno de Conda según las dependencias de Conda que se hayan especificado en la definición de entorno.

El segundo paso se omite si se especifican [dependencias administradas por el usuario](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment.pythonsection?view=azure-ml-py). En este caso, es responsable de instalar todos los paquetes de Python, ya sea al incluirlos en la imagen base o al especificar pasos de Docker personalizados en el primer paso. También es responsable de especificar la ubicación correcta para el archivo ejecutable de Python. También es posible usar una [imagen base de Docker personalizada](how-to-deploy-custom-docker-image.md).

### <a name="image-caching-and-reuse"></a>Almacenamiento en caché y reutilización de imágenes

Si usa la misma definición de entorno para otra ejecución, Machine Learning Service vuelve a usar la imagen almacenada en caché de la instancia de ACR del área de trabajo. 

Para ver los detalles de una imagen almacenada en caché, use el método [Environment.get_image_details](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment.environment?view=azure-ml-py#get-image-details-workspace-).

Para determinar si se debe reutilizar una imagen almacenada en caché o se debe crear una nueva, el servicio calcula [un valor hash](https://en.wikipedia.org/wiki/Hash_table) a partir de la definición de entorno y lo compara con los valores hash de los entornos existentes. El hash se basa en lo siguiente:
 
 * Valor de la propiedad de imagen base
 * Valor de la propiedad de pasos de Docker personalizados
 * Lista de los paquetes de Python en la definición de Conda
 * Lista de los paquetes en la definición de Spark 

El valor de hash no depende del nombre o de la versión del entorno; si cambia el nombre del entorno o crea uno con exactamente las mismas propiedades y paquetes de un entorno existente, el valor de hash seguirá siendo el mismo. Sin embargo, los cambios en la definición del entorno, como la adición o eliminación de un paquete de Python o el cambio de versión del paquete, hacen que el valor de hash cambie. Si se cambia el orden de las dependencias o de los canales de un entorno, se generará un nuevo entorno y, por tanto, será necesario volver a compilar la imagen. Es importante tener en cuenta que cualquier cambio en un entorno mantenido invalidará el valor de hash y dará como resultado un nuevo entorno "no mantenido".

El valor de hash calculado se compara con el de la instancia de ACR global y del área de trabajo (o con el del destino de proceso si se trata de ejecuciones locales). Si se encuentra una coincidencia, se extrae la imagen almacenada en caché; de lo contrario, se desencadena una compilación de la imagen. La duración de la extracción de una imagen almacenada en caché incluye el tiempo de descarga, mientras que la duración de la extracción de una imagen recién compilada incluye el tiempo de compilación y el tiempo de descarga. 

En el siguiente diagrama se muestran tres definiciones de entorno. Dos de ellas tienen nombres y versiones diferentes, pero la imagen base y los paquetes de Python son idénticos. Como tienen el mismo valor de hash, corresponden a la misma imagen almacenada en caché. El tercer entorno tiene diferentes paquetes y versiones de Python y, por lo tanto, corresponde a una imagen almacenada en caché distinta.

![Diagrama del almacenamiento en caché del entorno como imágenes de Docker](./media/concept-environments/environment-caching.png)

>[!IMPORTANT]
> Si crea un entorno con una dependencia de paquete desanclada (por ejemplo, ```numpy```), ese entorno seguirá usando la versión del paquete instalada _en el momento de la creación del entorno_. Además, cualquier entorno futuro con una definición coincidente seguirá usando la versión anterior. 

Para actualizar el paquete, especifique un número de versión para forzar la recompilación de la imagen; por ejemplo, ```numpy==1.18.1```. Se instalarán nuevas dependencias, incluidas las anidadas, que podrían estropear un escenario que anteriormente funcionaba. 

> [!WARNING]
>  El método [Environment.build](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment.environment?view=azure-ml-py#build-workspace--image-build-compute-none-) recompilará la imagen almacenada en caché, con el posible efecto secundario de que se actualicen los paquetes sin especificar y que todas las definiciones de entorno correspondientes a esa imagen almacenada en caché ya no puedan reproducirse.

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [crear y usar entornos](how-to-use-environments.md) en Azure Machine Learning.
* Vea la documentación de referencia del SDK de Python para la [clase environment](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment(class)?view=azure-ml-py).
* Vea la documentación de referencia del SDK de R para los [entornos](https://azure.github.io/azureml-sdk-for-r/reference/index.html#section-environments).
