---
title: 'Tutorial: Creación de un proyecto de etiquetado para la clasificación de imágenes'
titleSuffix: Azure Machine Learning
description: Aprenda a administrar el proceso de etiquetado de imágenes para que se puedan usar en modelos de clasificación de imágenes con varias clases.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.author: sgilley
author: sdgilley
ms.reviewer: ranku
ms.date: 04/09/2020
ms.openlocfilehash: 9e24a652bb4e577ff9b604b6b4f5284883723ee5
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906711"
---
# <a name="tutorial-create-a-labeling-project-for-multi-class-image-classification"></a>Tutorial: Creación de un proyecto de etiquetado para la clasificación de imágenes con varias clases 


En este tutorial se muestra cómo administrar el proceso de creación de etiquetas (también conocido como etiquetado) de imágenes que se usarán como datos para crear modelos de aprendizaje automático. El etiquetado de datos en Azure Machine Learning está en versión preliminar pública.

Si desea entrenar un modelo de aprendizaje automático para clasificar las imágenes, necesita cientos o incluso miles de imágenes que estén correctamente etiquetadas.  Azure Machine Learning le ayuda a administrar el progreso de su equipo privado de expertos en dominio a medida que etiquetan los datos.
 
En este tutorial, se usarán imágenes de gatos y perros.  Puesto que cada imagen es de un gato o de un perro, se trata de un proyecto de etiquetado *con varias clases*. Aprenderá a:

> [!div class="checklist"]
>
> * Crear una cuenta de almacenamiento de Azure y cargar imágenes en la cuenta.
> * Cree un área de trabajo de Azure Machine Learning.
> * Crear un proyecto de etiquetado de imágenes con varias clases.
> * Etiquetar los datos.  Tanto usted como sus etiquetadores pueden realizar esta tarea.
> * Finalizar el proyecto con la revisión y exportación de los datos.

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://aka.ms/AMLFree).

## <a name="create-a-workspace"></a>Crear un área de trabajo

Un área de trabajo de Azure Machine Learning es un recurso básico de la nube que se usa para experimentar, entrenar e implementar modelos de aprendizaje automático. Vincula la suscripción y el grupo de recursos de Azure con un objeto fácilmente consumido del servicio.

Puede crear un área de trabajo mediante Azure Portal, una consola basada en web para administrar los recursos de Azure.

[!INCLUDE [aml-create-portal](../../includes/aml-create-in-portal.md)]

## <a name="start-a-labeling-project"></a>Inicio de un proyecto de etiquetado

A continuación administrará el proyecto de etiquetado de datos en Azure Machine Learning Studio, una interfaz consolidada que incluye herramientas de aprendizaje automático para realizar escenarios de ciencia de datos para los profesionales de la misma con cualquier nivel de conocimientos. Studio no se admite en los exploradores Internet Explorer.

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com).

1. Seleccione la suscripción y el área de trabajo que ha creado.

### <a name="create-a-datastore"></a><a name="create-datastore"></a>Creación de un almacén de datos

Los almacenes de datos de Azure Machine Learning se usan para guardar la información de conexión, como el identificador de suscripción y la autorización de token. Aquí se usa un almacén de datos para conectarse a la cuenta de almacenamiento que contiene las imágenes necesarias para este tutorial.

1. En el lado izquierdo del área de trabajo, seleccione **Almacenes de datos**.

1. Seleccione **+ Nuevo almacén de datos**.

1. Rellene el formulario con estos valores:

    Campo|Descripción 
    ---|---
    Nombre del almacén de datos | Asigne un nombre al almacén de datos.  Aquí se usará **labeling_tutorial**.
    Tipo de almacén de datos | Seleccione el tipo de almacenamiento.  En este caso se utilizará **Azure Blob Storage**, el almacenamiento preferido para las imágenes.
    Método de selección de cuentas | Seleccione **Especificar manualmente**.
    URL | `https://azureopendatastorage.blob.core.windows.net/openimagescontainer`
    Tipo de autenticación | Seleccione **Token de SAS**.
    Clave de cuenta | `?sv=2019-02-02&ss=bfqt&srt=sco&sp=rl&se=2025-03-25T04:51:17Z&st=2020-03-24T20:51:17Z&spr=https&sig=7D7SdkQidGT6pURQ9R4SUzWGxZ%2BHlNPCstoSRRVg8OY%3D`

1. Seleccione **Crear** para crear el almacén de datos.

### <a name="create-a-labeling-project"></a>Crear un proyecto de etiquetado

Ahora que tiene acceso a los datos que desea que se etiqueten, cree el proyecto de etiquetado.

1. En la parte superior de la página, seleccione **Proyectos**.

1. Seleccione **+ Agregar proyecto**.

    :::image type="content" source="media/tutorial-labeling/create-project.png" alt-text="Creación de un proyecto":::

### <a name="project-details"></a>Detalles del proyecto

1. Use la entrada siguiente para el formulario **Detalles del proyecto**:

    Campo|Descripción 
    ---|---
    Nombre de proyecto | Asigne un nombre a su proyecto.  Aquí vamos a usar **tutorial-cats-n-dogs**.
    Etiquetado de tipos de tareas | Seleccione **Clasificación de imágenes con varias clases**.  
    
    Seleccione **Siguiente** para continuar con la creación del proyecto.

### <a name="select-or-create-a-dataset"></a>Selección o creación de un conjunto de datos

1.   En el formulario **Seleccione o cree un conjunto de datos**, seleccione la segunda opción, **Crear un conjunto de datos** y, a continuación, el vínculo **De almacén de datos**.

1. Use la siguiente entrada para el formulario **Creación de un conjunto de datos a partir de un almacén de datos**:

    1. En el formulario **Información básica**, agregue un nombre, aquí usaremos **images-for-tutorial**.  Si así lo desea, agregue una descripción.  Luego, seleccione **Siguiente**.
    1. En el formulario **Datastore selection** (Selección de almacén de datos), utilice la lista desplegable para seleccionar la opción **Previously created datastore** (Almacén de datos creado anteriormente), por ejemplo **tutorial_images (Azure Blob Storage)**
    1. A continuación, en el formulario **Datastore selection** (Selección de almacén de datos), seleccione **Examinar** y, a continuación, **MultiClass - DogsCats**.  Seleccione **Guardar** para usar **/MultiClass - DogsCats** como ruta de acceso.
    1. Seleccione **Siguiente** para confirmar los detalles y, a continuación, **Crear** para crear el conjunto de datos.
    1. Seleccione el círculo situado junto al nombre del conjunto de datos en la lista, por ejemplo **images-for-tutorial**.

1. Seleccione **Siguiente** para continuar con la creación del proyecto.

### <a name="incremental-refresh"></a>Actualización incremental

Si planea agregar nuevas imágenes al conjunto de datos, la actualización incremental las encontrará y las agregará a su proyecto.  Si se habilita esta característica, el proyecto buscará periódicamente nuevas imágenes.  No va a agregar nuevas imágenes al almacén de datos en este tutorial, por lo que debe dejar esta característica desactivada.

Seleccione **Next** (Siguiente) para continuar.

### <a name="label-classes"></a>Clases de etiquetas

1. En el formulario **Clases de etiquetas**, escriba un nombre de etiqueta y, a continuación, seleccione **+ Agregar etiqueta** para escribir la etiqueta siguiente.  En este proyecto, las etiquetas son **Cat**, **Dog** y **Uncertain** (Incierto).

1. Seleccione **Siguiente** cuando haya agregado todas las etiquetas.

### <a name="labeling-instructions"></a>Instrucciones de etiquetado

1. En el formulario **Instrucciones de etiquetado**, puede proporcionar un vínculo a un sitio web que da instrucciones detalladas para los etiquetadores.  Para este tutorial lo dejaremos en blanco.

1. También puede agregar una breve descripción de la tarea directamente en el formulario.  Escriba **Labeling tutorial - Cats & Dogs.**

1. Seleccione **Next** (Siguiente).

1. En la sección **Etiquetado con asistencia de ML**, deje la casilla desactivada. El etiquetado con asistencia de ML requiere más datos de los que va a usar en este tutorial.

1. Seleccione **Create project** (Crear proyecto).

Esta página no se actualiza automáticamente. Espere unos momentos y actualice la página manualmente hasta que el estado del proyecto cambie a **Creado**.

## <a name="start-labeling"></a>Inicio del etiquetado

Ahora ya ha configurado los recursos de Azure y un proyecto de etiquetado de datos. Ha llegado el momento de agregar etiquetas a los datos.

### <a name="tag-the-images"></a>Etiquetado de imágenes

En esta parte del tutorial, cambiará los roles de *administrador del proyecto* al de *etiquetador*.  Cualquier persona que tenga acceso de colaborador al área de trabajo puede convertirse en etiquetador.

1. En [Machine Learning Studio](https://ml.azure.com), seleccione **Etiquetado de datos** en el lado izquierdo para buscar el proyecto.  

1. Seleccione **Label link** (Etiqueta de vínculo) para el proyecto.

1. Lea las instrucciones y, luego, seleccione **Tareas**.

1. Seleccione una imagen en miniatura a la derecha para mostrar el número de imágenes que desea etiquetar en una sola operación. Para poder continuar tiene que etiquetar todas estas imágenes. Cambie el diseño solo cuando tenga una página nueva de datos sin etiquetar. El cambio de diseño borra el trabajo de etiquetado en curso de la página.

1. Seleccione una o varias imágenes y, a continuación, escoja una etiqueta para aplicarla a la selección. La etiqueta aparece debajo de la imagen.  Siga seleccionando y etiquetando todas las imágenes de la página.  Para seleccionar todas las imágenes mostradas al mismo tiempo, use **Seleccionar todo**. Seleccione al menos una imagen para aplicar una etiqueta.


    > [!TIP]
    > Puede seleccionar las nueve primeras etiquetas con las teclas numéricas del teclado.

1. Una vez que se etiqueten todas las imágenes de la página, seleccione **Enviar** para enviarlas.

    ![Etiquetado de imágenes](media/tutorial-labeling/catsndogs.gif)

1. Una vez que haya enviado etiquetas para los datos con los que está trabajando, Azure actualizará la página con un nuevo conjunto de imágenes de la cola de trabajo.

## <a name="complete-the-project"></a>Finalización del proyecto

Ahora cambiará los roles de nuevo a *administrador del proyecto* para el proyecto de etiquetado.

Como administrador, puede que desee revisar el trabajo de su etiquetador.  

### <a name="review-labeled-data"></a>Revisión de los datos etiquetados

1. En [Machine Learning Studio](https://ml.azure.com), seleccione **Etiquetado de datos** en el lado izquierdo para buscar el proyecto.  

1. Seleccione el vínculo con el nombre del proyecto.

1. El panel muestra el progreso del proyecto.

1. En la parte superior de la página, seleccione **Datos**.

1. En el lado izquierdo, seleccione **Labeled data** (Datos etiquetados) para ver las imágenes etiquetadas.  

1. Cuando no esté de acuerdo con una etiqueta, seleccione la imagen y, a continuación, seleccione **Rechazar** en la parte inferior de la página.  Se quitarán las etiquetas y la imagen se devolverá a la cola de imágenes sin etiquetar.

### <a name="export-labeled-data"></a>Exportación de los datos etiquetados

En cualquier momento, puede exportar los datos de etiquetas para realizar experimentos de Machine Learning. En lugar de esperar a que todas las imágenes estén etiquetadas, los usuarios suelen exportar varias veces y entrenar diferentes modelos.

Las etiquetas de imagen se pueden exportar en [formato COCO](http://cocodataset.org/#format-data) o como un conjunto de datos de Azure Machine Learning. El formato del conjunto de datos facilita su uso para el entrenamiento en Azure Machine Learning.  

1. En [Machine Learning Studio](https://ml.azure.com), seleccione **Etiquetado de datos** en el lado izquierdo para buscar el proyecto.  

1. Seleccione el vínculo con el nombre del proyecto.

1. Seleccione **Exportar** y elija **Export as Azure ML Dataset** (Exportar como conjunto de datos de Azure ML). 

    El estado de la exportación aparece justo debajo del botón **Exportar**. 

1. Una vez que las etiquetas se hayan exportado correctamente, seleccione **Conjuntos de datos** en el lado izquierdo para ver los resultados.

## <a name="clean-up-resources"></a>Limpieza de recursos


[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha etiquetado imágenes.  Ahora use los datos etiquetados:

> [!div class="nextstepaction"]
> [Entrenamiento de un modelo de reconocimiento de imágenes de aprendizaje automático](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datasets-tutorial/labeled-datasets/labeled-datasets.ipynb).
