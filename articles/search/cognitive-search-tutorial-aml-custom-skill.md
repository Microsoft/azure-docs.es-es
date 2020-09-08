---
title: 'Tutorial: Creación e implementación de una aptitud personalizada con Azure Machine Learning'
titleSuffix: Azure Cognitive Search
description: En este tutorial se muestra cómo usar Azure Machine Learning para compilar e implementar una aptitud personalizada para la canalización de enriquecimiento con IA de Azure Cognitive Search.
manager: nitinme
author: tchristiani
ms.author: terrychr
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 06/10/2020
ms.openlocfilehash: a4e686fe7adcc7e990a26484bc5850de977e862a
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88924595"
---
# <a name="tutorial-build-and-deploy-a-custom-skill-with-azure-machine-learning"></a>Tutorial: Creación e implementación de una aptitud personalizada con Azure Machine Learning 

En este tutorial, usará el [conjunto de datos de reseñas de hoteles](https://www.kaggle.com/datafiniti/hotel-reviews) (que se distribuye con la licencia de Creative Commons [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.txt)) para crear una [aptitud personalizada](./cognitive-search-aml-skill.md) mediante Azure Machine Learning y extraer opiniones basadas en aspectos de las reseñas. De esta forma, se podrá atribuir correctamente la asignación de opiniones positivas y negativas dentro de la misma reseña a las entidades identificadas, como personal, habitación, vestíbulo o piscina.

Para entrenar el modelo de opinión basada en aspectos de Azure Machine Learning, usará el [repositorio nlp-recipes](https://github.com/microsoft/nlp-recipes/tree/master/examples/sentiment_analysis/absa). A continuación, el modelo se implementará como un punto de conexión en un clúster de Azure Kubernetes. Una vez implementado, el punto de conexión se agrega a la canalización de enriquecimiento como una aptitud de Azure Machine Learning para su uso por parte del servicio Cognitive Search.

Se proporcionan dos conjuntos de datos. Si quiere entrenar el modelo por su cuenta, se necesita el archivo hotel_reviews_1000.csv. ¿Prefiere omitir el paso de entrenamiento? Descargue el archivo hotel_reviews_100.csv.

> [!div class="checklist"]
> * Creación de una instancia de Azure Cognitive Search
> * Creación de un área de trabajo de Azure Machine Learning (el servicio de búsqueda y el área de trabajo deben estar en la misma suscripción)
> * Entrenamiento e implementación de un modelo en un clúster de Azure Kubernetes
> * Vinculación de una canalización de enriquecimiento con IA al modelo implementado
> * Ingesta de la salida del modelo implementado como una aptitud personalizada

> [!IMPORTANT] 
> Esta aptitud está actualmente en versión preliminar pública. La funcionalidad de versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Actualmente no hay compatibilidad con el SDK de .NET.

## <a name="prerequisites"></a>Requisitos previos

* Suscripción de Azure: obtenga una [suscripción gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Servicio Cognitive Search](./search-get-started-arm.md)
* [Recurso de Cognitive Services](../cognitive-services/cognitive-services-apis-create-account.md?tabs=multiservice%2cwindows)
* [Cuenta de Azure Storage](../storage/common/storage-account-create.md?tabs=azure-portal&toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
* [área de trabajo de Azure Machine Learning](../machine-learning/how-to-manage-workspace.md)

## <a name="setup"></a>Configurar

* Clone o descargue el contenido del [repositorio de ejemplo](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill).
* Si la descarga es un archivo comprimido, extraiga el contenido. Asegúrese de que los archivos sean de lectura y escritura.
* Al configurar las cuentas y los servicios de Azure, copie los nombres y las claves en un archivo de texto de acceso fácil. Los nombres y las claves se agregarán a la primera celda del cuaderno, donde se definen las variables para acceder a los servicios de Azure.
* Si no está familiarizado con Azure Machine Learning y sus requisitos, revise estos documentos antes de empezar:
 * [Configuración de un entorno de desarrollo para Azure Machine Learning](../machine-learning/how-to-configure-environment.md)
 * [Creación y administración de áreas de trabajo de Azure Machine Learning en Azure Portal](../machine-learning/how-to-manage-workspace.md)
 * Al configurar el entorno de desarrollo para Azure Machine Learning, considere la posibilidad de usar la [instancia de proceso basada en la nube](../machine-learning/how-to-configure-environment.md#compute-instance) para comenzar de manera más fácil y rápida.
* Cargue el archivo del conjunto de datos en un contenedor de la cuenta de almacenamiento. Si quiere realizar el paso de entrenamiento en el cuaderno, el archivo más grande es necesario. Si prefiere omitir el paso de entrenamiento, se recomienda el archivo más pequeño.

## <a name="open-notebook-and-connect-to-azure-services"></a>Apertura del cuaderno y conexión a los servicios de Azure

1. Coloque toda la información necesaria de las variables que permitirán el acceso a los servicios de Azure dentro de la primera celda y ejecute la celda.
1. Al ejecutar la segunda celda, se confirmará que se ha conectado al servicio de búsqueda de la suscripción.
1. En las secciones 1.1 a 1.5 se creará el almacén de datos del servicio de búsqueda, el conjunto de aptitudes, el índice y el indexador.

En este momento puede optar por omitir los pasos para crear el conjunto de datos de entrenamiento y experimentar en Azure Machine Learning y pasar directamente a registrar los dos modelos que se proporcionan en la carpeta models del repositorio de GitHub. Si omite estos pasos, en el cuaderno pasará a la sección 3.5, en la que escribirá el script de puntuación. Este proceso ahorra tiempo; los pasos de descarga y carga de los datos pueden tardar hasta 30 minutos en completarse.

## <a name="creating-and-training-the-models"></a>Creación y entrenamiento de los modelos

La sección 2 tiene seis celdas donde se descarga el archivo de inserciones GloVe del repositorio npl-recipes. Después de la descarga, el archivo se carga en el almacén de datos de Azure Machine Learning. El archivo ZIP tiene unos 2 G y tardará algún tiempo en realizar estas tareas. Una vez cargado, los datos de entrenamiento se extraen y ahora está listo para pasar a la sección 3.

## <a name="train-the-aspect-based-sentiment-model-and-deploy-your-endpoint"></a>Entrenamiento del modelo de opinión basada en aspectos e implementación del punto de conexión

En la sección 3 del cuaderno se entrenarán los modelos creados en la sección 2, se registrarán esos modelos y se implementarán como un punto de conexión en un clúster de Azure Kubernetes. Si no está familiarizado con Azure Kubernetes, se recomienda encarecidamente que revise los artículos siguientes antes de intentar crear un clúster de inferencia:

* [Azure Kubernetes Service (AKS)](../aks/intro-kubernetes.md)
* [Conceptos básicos de Kubernetes de Azure Kubernetes Service (AKS)](../aks/concepts-clusters-workloads.md)
* [Cuotas, restricciones de tamaño de máquinas virtuales y disponibilidad de regiones en Azure Kubernetes Service (AKS)](../aks/quotas-skus-regions.md)

La creación e implementación del clúster de inferencia puede tardar hasta 30 minutos. Se recomienda probar el servicio web antes de continuar con los pasos finales, de forma que se actualice el conjunto de aptitudes y se ejecute el indexador.

## <a name="update-the-skillset"></a>Actualización del conjunto de aptitudes

La sección 4 del cuaderno tiene cuatro celdas donde se actualizan el conjunto de aptitudes y el indexador. Como alternativa, puede usar el portal para seleccionar y aplicar la nueva aptitud al conjunto de aptitudes y, luego, ejecutar el indexador para actualizar el servicio de búsqueda.

> [!VIDEO https://channel9.msdn.com/Shows/AI-Show/Active-Learning-with-Azure-Cognitive-Search/player#time=19m35s:paused/03/player]

En el portal, vaya al conjunto de aptitudes y seleccione el vínculo de definición del conjunto de aptitudes (JSON). El portal mostrará el archivo JSON del conjunto de aptitudes que se creó en las primeras celdas del cuaderno. A la derecha de la pantalla hay un menú desplegable en el que puede seleccionar la plantilla de definición de aptitudes. Seleccione la plantilla de Azure Machine Learning (AML). Proporcione el nombre del área de trabajo de Azure ML y el punto de conexión del modelo implementado en el clúster de inferencia. La plantilla se actualizará con el URI y la clave del punto de conexión.

> [!div class="mx-imgBorder"]
> ![Plantilla de definición del conjunto de aptitudes](media/cognitive-search-aml-skill/portal-aml-skillset-definition.png)

Copie la plantilla del conjunto de aptitudes de la ventana y péguela en la definición del conjunto de aptitudes de la izquierda. Edite la plantilla para proporcionar los valores que faltan en:

* Nombre
* Descripción
* Context
* "entradas": nombre y origen
* "salidas": nombre y targetName

Guarde el conjunto de aptitudes.

A continuación, vaya al indexador y seleccione el vínculo de definición del indexador (JSON). El portal mostrará el archivo JSON del indexador que se creó en las primeras celdas del cuaderno. Las asignaciones de campos de salida deberán actualizarse con asignaciones de campos adicionales para garantizar que el indexador pueda administrarlas y pasarlas correctamente. Guarde los cambios y, luego, seleccione Ejecutar. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede encontrar y administrar recursos en el portal, mediante el vínculo **Todos los recursos** o **Grupos de recursos** en el panel de navegación izquierdo.

Si está usando un servicio gratuito, recuerde que está limitado a tres índices, indexadores y orígenes de datos. Puede eliminar elementos individuales en el portal para mantenerse por debajo del límite.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Revise la API web de aptitud personalizada](./cognitive-search-custom-skill-web-api.md)
> [Más información sobre cómo agregar aptitudes personalizadas a la canalización de enriquecimiento](./cognitive-search-custom-skill-interface.md)