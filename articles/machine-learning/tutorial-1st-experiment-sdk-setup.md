---
title: 'Tutorial: Introducción a los cuadernos de Jupyter Notebook (Python)'
titleSuffix: Azure Machine Learning
description: Tutoriales para la configuración de Jupyter Notebook.  Cree un área de trabajo de Azure Machine Learning, clone los cuadernos de Jupyter Notebook en el área de trabajo y cree una instancia de proceso en la que se ejecuten los cuadernos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: sdgilley
ms.author: sgilley
ms.date: 02/10/2020
ms.custom: devx-track-python
ms.openlocfilehash: de52013628f5d02bedcf72a99e0fad25cabe5d8f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90896865"
---
# <a name="tutorial-get-started-with-azure-machine-learning-in-jupyter-notebooks"></a>Tutorial: Introducción a Azure Machine Learning en cuadernos de Jupyter Notebook

En este tutorial, completará los pasos para empezar a trabajar con Azure Machine Learning mediante cuadernos de Jupyter Notebook en una [estación de trabajo (instancia de proceso) administrada basada en la nube](concept-compute-instance.md). Este tutorial es un precursor de todos los demás tutoriales de Jupyter Notebook.

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Crear un [área de trabajo de Azure Machine Learning](concept-workspace.md) para usarla en otros tutoriales de Jupyter Notebook.
> * Clone el cuaderno de tutoriales en su carpeta en el área de trabajo.
> * Cree una instancia de proceso basada en la nube que tenga instalado y preconfigurado el SDK de Python para Azure Machine Learning.

Si no tiene una suscripción a Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

## <a name="create-a-workspace"></a>Crear un área de trabajo

Un área de trabajo de Azure Machine Learning es un recurso básico de la nube que se usa para experimentar, entrenar e implementar modelos de aprendizaje automático. Vincula la suscripción y el grupo de recursos de Azure con un objeto fácilmente consumido del servicio.

Puede crear un área de trabajo mediante Azure Portal, una consola basada en web para administrar los recursos de Azure.

[!INCLUDE [aml-create-portal](../../includes/aml-create-in-portal.md)]

>[!IMPORTANT]
> Tome nota del **área de trabajo** y de la **suscripción**. Los necesitará para asegurarse de que crea el experimento en el lugar correcto. 

## <a name="run-notebook-in-your-workspace"></a><a name="azure"></a>Ejecución de un cuaderno en el área de trabajo

Azure Machine Learning incluye un servidor de cuadernos en la nube del área de trabajo para obtener una experiencia sin instalación y configurada previamente. Si prefiere tener control sobre su entorno, los paquetes y las dependencias, use [su propio entorno](tutorial-1st-experiment-sdk-setup-local.md).

 Siga este vídeo o use los pasos detallados que se indican a continuación para clonar y ejecutar el cuaderno del tutorial desde el área de trabajo.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4mTUr]

### <a name="clone-a-notebook-folder"></a>Clonación de la carpeta de un cuaderno

Complete los siguientes pasos de configuración y ejecución del experimento en Azure Machine Learning Studio, una interfaz consolidada que incluye herramientas de aprendizaje automático para realizar escenarios de ciencia de datos para los profesionales de ciencia de datos de todos los niveles de conocimiento.

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/).

1. Seleccione la suscripción y el área de trabajo que ha creado.

1. Seleccione **Notebooks** en la parte izquierda.

1. Seleccione la pestaña **Ejemplos** en la parte superior.

1. Abra la carpeta **Python**.

1. Abra la carpeta con un número de versión.  Este número representa la versión actual del SDK de Python.

1. Seleccione **"..."** a la derecha de la carpeta **tutorials** (tutoriales) y seleccione **Clone** (Clonar).

    :::image type="content" source="media/tutorial-1st-experiment-sdk-setup/clone-tutorials.png" alt-text="Clonación de la carpeta tutorials":::

1. Aparece una lista de las carpetas que muestran los usuarios con acceso al área de trabajo.  Seleccione la carpeta donde se va a clonar la carpeta **tutorials**.

### <a name="open-the-cloned-notebook"></a><a name="open"></a>Apertura del cuaderno clonado

1. Abra la carpeta **tutorials** que se acaba de cerrar en la sección **Archivos de usuario**.

    > [!IMPORTANT]
    > Puede ver los cuadernos en la carpeta **samples** (ejemplos), pero no puede ejecutar un cuaderno desde aquí.  Para ejecutar un cuaderno, asegúrese de que abre la versión clonada del cuaderno en la sección **User Files** (Archivos de usuario).
    
1. Seleccione el archivo **tutorial-1st-experiment-sdk-train.ipynb** de la carpeta **tutorials/image-classification-mnist-data**.

    :::image type="content" source="media/tutorial-1st-experiment-sdk-setup/expand-user-folder.png" alt-text="Apertura de la carpeta tutorials":::

1. En la barra superior, seleccione una instancia de proceso para usarla para ejecutar el cuaderno. Estas máquinas virtuales están configuradas previamente con [todo lo necesario para ejecutar Azure Machine Learning](concept-compute-instance.md#contents).

1. Si no encuentra ninguna máquina virtual, seleccione **+ Add** (+ Agregar) para crear la máquina virtual de instancia de proceso. 

    1. Al crear una máquina virtual, siga estas reglas:  
        + El nombre es obligatorio y no puede estar vacío.
        + Debe ser único (no distingue mayúsculas de minúsculas) en todas las instancias de proceso existentes en la región de Azure del área de trabajo o la instancia de proceso. En caso contrario, recibirá una alerta.
        + Caracteres válidos son letras mayúsculas y minúsculas, números (del 0 al 9) y guiones (-).
        + Debe contener entre 3 y 24 caracteres.
        + Debe empezar por una letra (no un número ni un guion).
        + Si se usa el guion, debe ir seguido al menos de una letra. Ejemplo: Test-, test-0, test-01 no son válidos, mientras que test-a0 y test-0a son ejemplos válidos.

    1.  Seleccione el tamaño de la máquina virtual de entre las opciones disponibles. En los tutoriales, la máquina virtual predeterminada es una buena elección.

    1. Seleccione **Crear**. La configuración de la máquina virtual puede tardar aproximadamente 5 minutos.

1. Una vez disponible, se mostrará en la barra de herramientas superior.  Ahora puede ejecutar el cuaderon con **Run all** (Ejecutar todo) de la barra de herramientas o **Mayús + Entrar** en las celdas de código del cuaderno.

Si tiene widgets personalizados o prefiere usar Jupyter/JupyterLab, seleccione la lista desplegable **Jupyter** en el extremo derecho y, después, seleccione **Jupyter** o **JupyterLab**. Se abrirá la nueva ventana del explorador.

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene un entorno de desarrollo configurado, continúe con el entrenamiento de un modelo en un cuaderno de Jupyter Notebook:

> [!div class="nextstepaction"]
> [Tutorial: Entrenamiento de modelos de clasificación de imágenes con los datos MNIST y scikit-learn](tutorial-train-models-with-aml.md)

<a name="stop-compute-instance"></a> Si no pretende seguir ningún otro tutorial ahora, debe detener la máquina virtual del servidor de cuadernos en la nube cuando no se use para reducir el costo.

[!INCLUDE [aml-stop-server](../../includes/aml-stop-server.md)]
