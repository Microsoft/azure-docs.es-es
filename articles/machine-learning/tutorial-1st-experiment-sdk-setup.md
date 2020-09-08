---
title: 'Tutorial: Experimento en cuadernos de Jupyter Notebook (Python)'
titleSuffix: Machine Learning - Azure
description: En este tutorial empezará a trabajar con el SDK de Python para Azure Machine Learning que se ejecuta en cuadernos de Jupyter.  En la parte 1 se crea un área de trabajo en la que se administran los experimentos y los modelos de Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: sdgilley
ms.author: sgilley
ms.date: 02/10/2020
ms.custom: devx-track-python
ms.openlocfilehash: ff23a42d9b96b8411d8b2f82ab8303e2a8a69953
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852725"
---
# <a name="tutorial-get-started-creating-your-first-ml-experiment-with-the-python-sdk"></a>Tutorial: Creación del primer experimento de ML con el SDK de Python
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este tutorial, realizará los pasos de un extremo a otro para empezar a trabajar con el SDK de Python para Azure Machine Learning que se ejecuta en cuadernos de Jupyter. Este tutorial es la **parte uno de una serie de tutoriales de dos partes**, y abarca la instalación y la configuración del entorno de Python, así como la creación de un área de trabajo para administrar los experimentos y los modelos de aprendizaje automático. En la [**segunda parte**](tutorial-1st-experiment-sdk-train.md) se usa este área de trabajo para entrenar varios modelos de aprendizaje automático e introducir el proceso de administración de modelos mediante Azure Machine Learning Studio y el SDK.

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Crear un [área de trabajo de Azure Machine Learning](concept-workspace.md) para usarla en el siguiente tutorial.
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

En este tutorial se usa el servidor de cuadernos en la nube del área de trabajo para obtener una experiencia sin instalación y configurada previamente. Si prefiere tener control sobre su entorno, los paquetes y las dependencias, use [su propio entorno](how-to-configure-environment.md#local).

 Siga este vídeo o use los pasos detallados que se indican a continuación para clonar y ejecutar el tutorial desde el área de trabajo. 

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
    
1. Seleccione el archivo **tutorial-1st-experiment-sdk-train.ipynb** de la carpeta **tutorials/create-first-ml-experiment**.

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

En este tutorial ha completado estas tareas:

* Ha creado un área de trabajo de Azure Machine Learning.
* Ha creado y configurado un servidor de cuadernos en la nube en el área de trabajo.

En la **segunda parte** del tutorial, se ejecuta el código en `tutorial-1st-experiment-sdk-train.ipynb` para entrenar un modelo de Machine Learning. 

> [!div class="nextstepaction"]
> [Tutorial: Entrenamiento del primer modelo](tutorial-1st-experiment-sdk-train.md)

> [!IMPORTANT]
> Si no está pensando en la segunda parte de este tutorial o en otros tutoriales, debe [detener la máquina virtual del servidor de cuadernos en la nube](tutorial-1st-experiment-sdk-train.md#clean-up-resources) cuando no se use para reducir el costo.


