---
title: Ejecución de los cuadernos de Jupyter Notebook en el área de trabajo
titleSuffix: Azure Machine Learning
description: Aprenda a ejecutar un cuaderno de Jupyter Notebook sin dejar el área de trabajo en Azure Machine Learning Studio.
services: machine-learning
author: abeomor
ms.author: osomorog
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.date: 06/27/2020
ms.openlocfilehash: b310f195477f4cba83ef30c6fb5ea2fd24c4df9a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90897380"
---
# <a name="how-to-run-jupyter-notebooks-in-your-workspace"></a>Ejecución de los cuadernos de Jupyter Notebook en el área de trabajo


Aprenda a ejecutar sus cuadernos de Jupyter Notebook directamente en el área de trabajo en Azure Machine Learning Studio. Aunque puede iniciar [Jupyter](https://jupyter.org/) o [JupyterLab](https://jupyterlab.readthedocs.io), también puede editar y ejecutar los cuadernos sin tener que salir del área de trabajo.

Vea cómo puede:

* Crear cuadernos de Jupyter Notebook en su área de trabajo.
* Ejecutar un experimento desde un cuaderno.
* Cambiar el entorno del cuaderno.
* Buscar los detalles de las instancias de proceso utilizadas para ejecutar los cuadernos.

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://aka.ms/AMLFree) antes de empezar.
* Un área de trabajo de Machine Learning. Consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

## <a name="create-notebooks"></a><a name="create"></a> Creación de cuadernos

En el área de trabajo de Azure Machine Learning, cree un nuevo cuaderno de Jupyter Notebook y empiece a trabajar. El cuaderno recién creado se almacena en el almacenamiento predeterminado del área de trabajo. Este cuaderno se puede compartir con cualquier persona que tenga acceso al área de trabajo. 

Para crear un nuevo cuaderno: 

1. En [Azure Machine Learning Studio](https://ml.azure.com), abra el área de trabajo.
1. En el lado izquierdo, seleccione **Notebooks**. 
1. Seleccione el icono **Crear archivo** situado encima de la lista **Archivos de usuario** en la sección **Mis archivos**.

    :::image type="content" source="media/how-to-run-jupyter-notebooks/create-new-file.png" alt-text="Crear un nuevo archivo":::

1. Dele un nombre al archivo. 
1. En el caso de los archivos de Jupyter Notebook, seleccione **Notebook** como tipo de archivo.
1. Seleccione un directorio de archivos.
1. Seleccione **Crear**.

También puede crear archivos de texto.  Seleccione **Texto** como el tipo de archivo y agregue la extensión al nombre (por ejemplo, miarchivo.py o miarchivo.txt).  

También puede cargar carpetas y archivos, incluidos cuadernos, con las herramientas que se encuentra en la parte superior de la página Notebooks.  Los cuadernos y la mayoría de los tipos de archivo de texto se muestran en la sección de vista previa.  Para la mayoría de los demás tipos de archivo la vista previa no está disponible.

> [!IMPORTANT]
> El contenido de los cuadernos y scripts puede leer los datos de las sesiones y acceder a los datos sin su organización en Azure.  Cargar solo archivos de fuentes de confianza. Para más información, vea [Prácticas recomendadas de código seguro](concept-secure-code-best-practice.md#azure-ml-studio-notebooks).

### <a name="clone-samples"></a>Clonación de ejemplos

El área de trabajo contiene una carpeta de **ejemplos** con cuadernos diseñados para ayudarle a explorar el SDK y servir como ejemplo para sus propios proyectos de aprendizaje automático.  Puede clonar estos cuadernos en su propia carpeta en el contenedor de almacenamiento del área de trabajo.  

Para obtener un ejemplo, vea [Tutorial: Creación del primer experimento de Machine Learning](tutorial-1st-experiment-sdk-setup.md#azure).

### <a name="use-files-from-git-and-version-my-files"></a><a name="terminal"></a> Uso de los archivos de Git y versión Mis archivos

Puede acceder a todas las operaciones de Git mediante una ventana de terminal. Todos los archivos y carpetas de Git se almacenarán en el sistema de archivos del área de trabajo.

> [!NOTE]
> Agregue los archivos y las carpetas en cualquier parte dentro de la carpeta **~/cloudfiles/code/Users** para que estén visibles en todos los entornos de Jupyter.

Para tener acceso al terminal:

1. En [Azure Machine Learning Studio](https://ml.azure.com), abra el área de trabajo.
1. En el lado izquierdo, seleccione **Notebooks**.
1. Seleccione cualquier cuaderno situado en la sección **Archivos de usuario** en el lado izquierdo.  Si allí no tiene ningún cuaderno, primero [cree un cuaderno](#create).
1. Seleccione un destino de **Proceso** o cree uno nuevo y espere hasta que se esté ejecutando.
1. Seleccione el icono de **Abrir terminal**.

    :::image type="content" source="media/how-to-run-jupyter-notebooks/open-terminal.png" alt-text="Abra el terminal":::.

1. Si no ve el icono, seleccione los puntos **...** que se encuentran a la derecha del destino de proceso y, a continuación, seleccione **Abrir terminal**.

    :::image type="content" source="media/how-to-run-jupyter-notebooks/alt-open-terminal.png" alt-text="Abrir el terminal desde los puntos ...":::.


Obtenga más información sobre [la clonación de repositorios de GIT de en el sistema de archivos del área de trabajo](concept-train-model-git-integration.md#clone-git-repositories-into-your-workspace-file-system).


### <a name="share-notebooks-and-other-files"></a>Compartición de cuadernos y otros archivos

Copie y pegue la dirección URL para compartir un bloc de notas o un archivo.  Solo otros usuarios del área de trabajo pueden acceder a esta dirección URL.  Obtenga más información sobre la [concesión de acceso a su área de trabajo](how-to-assign-roles.md).

## <a name="edit-a-notebook"></a>Edición de un cuaderno

Para editar un cuaderno, abra cualquiera que esté situado en la sección **Archivos de usuario** de su área de trabajo. Haga clic en la celda que desee editar. 

Puede editar el cuaderno sin necesidad de conectarse a una instancia de proceso.  Cuando desee ejecutar las celdas en el cuaderno, seleccione o cree una instancia de proceso.  Si selecciona una instancia de proceso detenida, se iniciará automáticamente al ejecutar la primera celda.

Cuando se ejecuta una instancia de proceso, también puede usar la finalización de código, con la tecnología [IntelliSense](https://code.visualstudio.com/docs/editor/intellisense), en cualquier Python Notebook.

También puede iniciar Jupyter o JupyterLab desde la barra de herramientas de Notebook.  Azure Machine Learning no proporciona actualizaciones ni corrige errores de Jupyter o JupyterLab, ya que son productos de código abierto fuera de los límites del servicio de soporte técnico de Microsoft.

### <a name="use-intellisense"></a>Usar IntelliSense

[IntelliSense](https://code.visualstudio.com/docs/editor/intellisense) es una ayuda de finalización de código que incluye una serie de características: enumerar miembros, información de parámetros, información rápida y completar palabra. Estas características le ayudan a obtener más información sobre el código que está usando, realizar un seguimiento de los parámetros que está escribiendo y agregar llamadas a propiedades y métodos con solo unas cuantas pulsaciones de tecla.  

Al escribir código, use Ctrl + barra espaciadora para desencadenar IntelliSense.

### <a name="clean-your-notebook-preview"></a>Limpieza del cuaderno (versión preliminar)

> [!IMPORTANT]
> La característica de recopilación actualmente está en versión preliminar pública.
> Se ofrece la versión preliminar sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Durante la creación de un cuaderno, normalmente acaba con las celdas usadas para la exploración o la depuración de datos. La característica de *recopilación* le ayudará a crear un cuaderno limpio sin estas celdas extrañas.

1. Ejecute todas las celdas del cuaderno.
1. Seleccione la celda que contiene el código que desea que ejecute el nuevo cuaderno. Por ejemplo, el código que envía un experimento o quizás el código que registra un modelo.
1. Seleccione el icono de **recopilación** que aparece en la barra de herramientas de celda.
    :::image type="content" source="media/how-to-run-jupyter-notebooks/gather.png" alt-text="Captura de pantalla: seleccione el icono de recopilación":::
1. Escriba el nombre del nuevo cuaderno "recopilado".  

El nuevo cuaderno solo contiene celdas de código, con todas las celdas necesarias para generar los mismos resultados que la celda seleccionada para la recopilación.

### <a name="save-and-checkpoint-a-notebook"></a>Guardar y revisar un cuaderno

Azure Machine Learning crea un archivo de punto de comprobación cuando se crea un archivo  *ipynb* .

En la barra de herramientas del cuaderno, seleccione el menú y luego, **Archivo&gt;Guardar y punto de control** para guardar manualmente el cuaderno y agregará un archivo de punto de control asociado con el cuaderno.

:::image type="content" source="media/how-to-run-jupyter-notebooks/file-save.png" alt-text="Captura de pantalla de la herramienta guardar en la barra de herramientas del cuaderno":::

Cada cuaderno se guarda automáticamente cada 30 segundos. El guardado automático actualiza solo el archivo inicial  *ipynb* , no el archivo de punto de control.
 
Seleccione **Puntos de control** en el menú del cuaderno para crear un punto de control con nombre y revertir el cuaderno a un punto de control guardado.


### <a name="useful-keyboard-shortcuts"></a>Métodos abreviados de teclado útiles

|Teclado  |Acción  |
|---------|---------|
|Mayús+Entrar     |  Ejecución de una celda       |
|Ctrl + barra espaciadora | Activar IntelliSense |
|Ctrl + M (Windows)     |  Habilitación o deshabilitación de captura de pestañas en Notebook.       |
|Ctrl + Mayús + M (Mac & Linux)     |    Habilitación o deshabilitación de captura de pestañas en Notebook.     |
|Tab (cuando está habilitada la captura de pestañas) | Incorporación de un carácter '\t' (sangría)
|Tab (cuando está deshabilitada la captura de pestañas) | Cambio del enfoque al siguiente elemento enfocable (botón Eliminar celda, botón Ejecutar, etc.)

## <a name="delete-a-notebook"></a>Eliminación de un cuaderno

Los cuadernos de **ejemplo** *no* se pueden eliminar.  Estos cuadernos forman parte de Studio y se actualizan cada vez que se publica un nuevo SDK.  

Los **archivos de usuario** y los cuadernos *se pueden* eliminar de cualquiera de estas formas:

* En Studio, seleccione los puntos **...** al final de una carpeta o un archivo.  Asegúrese de usar un explorador compatible (Microsoft Edge, Chrome o Firefox).
* En la barra de herramientas de cualquier cuaderno de Notebook, seleccione [**Abrir terminal**](#terminal) para acceder a la ventana de terminal de la instancia de proceso.
* En Jupyter o JupyterLab con sus herramientas.

## <a name="run-an-experiment"></a>Ejecución de un experimento

Para ejecutar un experimento desde un cuaderno de Notebook, primero tiene que conectarse a una [instancia de proceso](concept-compute-instance.md) en ejecución. Si no tiene una instancia de proceso, siga estos pasos para crear una: 

1. Seleccione **+** en la barra de herramientas de Notebook. 
2. Asigne un nombre al proceso y elija el **Tamaño de la máquina virtual**. 
3. Seleccione **Crear**.
4. La instancia de proceso se conecta automáticamente al cuaderno de Notebook y ya puede ejecutar las celdas.

Solo puede ver y usar las instancias de proceso que usted cree.  Los **Archivos de usuario** se almacenan de forma independiente de la máquina virtual y se comparten entre todas las instancias de proceso en el área de trabajo.

### <a name="view-logs-and-output"></a>Ver registros y resultados

Utilice los [widgets de Notebook](https://docs.microsoft.com/python/api/azureml-widgets/azureml.widgets?view=azure-ml-py&preserve-view=true) para ver el progreso de la ejecución y los registros. Un widget es asincrónico y proporciona actualizaciones hasta que finaliza el entrenamiento. Los widgets de Azure Machine Learning también se admiten en Jupyter y JupterLab.

## <a name="change-the-notebook-environment"></a>Cambio del entorno del cuaderno.

La barra de herramientas de Notebook permite cambiar el entorno en el que se ejecuta el cuaderno.  

Estas acciones no cambiarán el estado del cuaderno o los valores de las variables del mismo:

|Acción  |Resultado  |
|---------|---------| --------|
|Detención del kernel     |  Detiene cualquier celda en ejecución. Al ejecutar una celda, se reiniciará automáticamente el kernel. |
|Desplazamiento a otra sección del área de trabajo     |     Se detienen las celdas en ejecución. |

Estas acciones restablecerán el estado del cuaderno y todas las variables del mismo.

|Acción  |Resultado  |
|---------|---------| --------|
| Cambio del kernel | Notebook usa un nuevo kernel |
| Cambio de proceso    |     Notebook utiliza automáticamente el nuevo proceso. |
| Restablecimiento del proceso | Se inicia de nuevo cuando se intenta ejecutar una celda. |
| Detención del proceso     |    No se ejecutarán las celdas.  |
| Apertura del cuaderno en Jupyter o JupyterLab     |    Notebook se abre en una nueva pestaña.  |

### <a name="add-new-kernels"></a>Incorporación de nuevos kernels

El cuaderno de Notebook buscará automáticamente todos los kernels de Jupyter instalados en la instancia de proceso conectada.  Para agregar un kernel a la instancia de proceso:

1. Seleccione [**Abrir terminal**](#terminal) en la barra de herramientas de Notebook.
1. Use la ventana de terminal para crear un nuevo entorno.  Por ejemplo, el código siguiente crea `newenv`:
    ```shell
    conda create --name newenv
    ```
1. Active el entorno.  Por ejemplo, después de crear `newenv`:

    ```shell
    conda activate newenv
    ```
1. Instalación del paquete PIP e ipykernel en el entorno nuevo y creación de un kernel para ese entorno conda

    ```shell
    conda install pip
    conda install ipykernel
    python -m ipykernel install --user --name newenv --display-name "Python (newenv)"
    ```

Se puede instalar cualquiera de los [kernels de Jupyter](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels) disponibles.

### <a name="status-indicators"></a>Indicadores de estado

Un indicador situado junto al elemento de la lista desplegable **Proceso** muestra su estado.  El estado también se muestra en el mismo elemento de la lista desplegable.  

|Color |Estado de proceso |
|---------|---------| 
| Verde | Proceso en ejecución |
| Rojo |Error de proceso | 
| Negro | Proceso detenido |
|  Azul claro |Proceso creando, iniciando, reiniciando, configurando |
|  Gris |Proceso eliminando, deteniendo |

Un indicador situado junto al elemento de lista desplegable **Kernel** muestra su estado.

|Color |Estado del kernel |
|---------|---------|
|  Verde |Kernel conectado, inactivo, ocupado|
|  Gris |Kernel no conectado |

## <a name="find-compute-details"></a>Detalles de proceso 

Busque los detalles sobre las instancias de proceso en la página **Proceso** de [Studio](https://ml.azure.com).

## <a name="next-steps"></a>Pasos siguientes

* [Ejecución de su primer experimento](tutorial-1st-experiment-sdk-train.md)
* [Copia de seguridad del almacenamiento de archivos con instantáneas](../storage/files/storage-snapshots-files.md)
