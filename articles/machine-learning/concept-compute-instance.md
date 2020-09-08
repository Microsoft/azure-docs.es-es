---
title: ¿Qué es una instancia de proceso de Azure Machine Learning?
titleSuffix: Azure Machine Learning
description: Más información sobre la instancia de proceso Azure Machine Learning, una estación de trabajo totalmente administrada basada en la nube.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sgilley
author: sdgilley
ms.date: 07/27/2020
ms.openlocfilehash: c72777bf2a4415a7f773f82a21a121f5e58f2ec0
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88651922"
---
# <a name="what-is-an-azure-machine-learning-compute-instance"></a>¿Qué es una instancia de proceso de Azure Machine Learning?

Una instancia de proceso de Azure Machine Learning es una estación de trabajo administrada basada en la nube para científicos de datos.

Las instancias de proceso permiten que sea fácil comenzar a desarrollar con Azure Machine Learning y proporcionan funcionalidades de administración y preparación empresarial para los administradores de TI.  

Use una instancia de proceso como el entorno de desarrollo completamente configurado y administrado en la nube para el aprendizaje automático. También se pueden usar como destino de proceso para el entrenamiento y la inferencia con fines de desarrollo y pruebas.  

Para el entrenamiento del modelo de calidad de producción, use un [clúster de proceso de Azure Machine Learning](how-to-set-up-training-targets.md#amlcompute) con capacidades de escalado de varios nodos. Para la implementación del modelo de calidad de producción, use el [clúster de Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md).

## <a name="why-use-a-compute-instance"></a>¿Por qué usar una instancia de proceso?

Una instancia de proceso es una estación de trabajo totalmente administrada basada en la nube optimizada para el entorno de desarrollo de aprendizaje automático. Proporciona las siguientes ventajas:

|Ventajas principales|Descripción|
|----|----|
|Productividad|Puede compilar e implementar modelos con cuadernos integrados y las siguientes herramientas en Azure Machine Learning Studio:<br/>-  Jupyter<br/>-  JupyterLab<br/>-RStudio (versión preliminar)<br/>La instancia de proceso está totalmente integrada con el área de trabajo y estudio de Azure Machine Learning. Puede compartir cuadernos y datos con otros científicos de datos en el área de trabajo. También puede configurar el desarrollo remoto VS Code con [SSH](how-to-set-up-vs-code-remote.md) |
|Administrada y segura|Reduzca la superficie de seguridad y agregue compatibilidad con los requisitos de seguridad empresariales. Las instancias de proceso proporcionan directivas de administración sólidas y configuraciones de red seguras, como:<br/><br/>- Aprovisionamiento automatizado a partir de plantillas de Resource Manager o del SDK de Azure Machine Learning<br/>- [Control de acceso basado en rol de Azure (RBAC de Azure)](/azure/role-based-access-control/overview)<br/>- [Compatibilidad con redes virtuales](how-to-enable-virtual-network.md#compute-instance)<br/>- Directiva SSH para habilitar o deshabilitar el acceso SSH<br/>TLS 1.2 habilitado |
|Preconfigurado &nbsp;para&nbsp;ML|Ahorre tiempo en las tareas de configuración con paquetes de ML preconfigurados y actualizados, marcos de aprendizaje profundo y controladores de GPU.|
|Totalmente personalizable|La amplia compatibilidad con los tipos de máquina virtual de Azure, incluidas las GPU y la personalización de bajo nivel persistente, como la instalación de paquetes y controladores, hace que los escenarios avanzados sean muy sencillos. |

## <a name="tools-and-environments"></a><a name="contents"></a>Herramientas y entornos

> [!IMPORTANT]
> Las herramientas marcadas (versión preliminar) a continuación se encuentran actualmente en versión preliminar pública.
> Se ofrece la versión preliminar sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

La estancia de proceso Azure Machine Learning permite crear, formar e implementar modelos en una experiencia de cuaderno completamente integrada en el área de trabajo.

Las siguientes herramientas y entornos están instalados en la instancia de proceso: 

|Herramientas y entornos generales|Detalles|
|----|:----:|
|Controladores|`CUDA`</br>`cuDNN`</br>`NVIDIA`</br>`Blob FUSE` |
|Intel MPI Library||
|Azure CLI ||
|Ejemplos de Azure Machine Learning ||
|Docker||
|Nginx||
|NCCL 2.0 ||
|Protobuf|| 

|Herramientas y entornos de **R**|Detalles|
|----|:----:|
|Edición de código abierto de RStudio Server (versión preliminar)||
|R kernel||
|SDK de Azure Machine Learning para R|[azuremlsdk](https://azure.github.io/azureml-sdk-for-r/reference/index.html)</br>Ejemplos del SDK|

|Herramientas y entornos de **PYTHON**|Detalles|
|----|----|
|Anaconda Python||
|Jupyter y extensiones||
|Jupyterlab y extensiones||
[SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py)</br>desde PyPI|Incluye la mayoría de los paquetes adicionales de azureml.  Para ver la lista completa, [abra una ventana de terminal en la instancia de proceso](how-to-run-jupyter-notebooks.md#terminal) y ejecute <br/> `conda list -n azureml_py36 azureml*` |
|Otros paquetes de PyPI|`jupytext`</br>`tensorboard`</br>`nbconvert`</br>`notebook`</br>`Pillow`|
|Paquetes Conda|`cython`</br>`numpy`</br>`ipykernel`</br>`scikit-learn`</br>`matplotlib`</br>`tqdm`</br>`joblib`</br>`nodejs`</br>`nb_conda_kernels`|
|Paquetes de aprendizaje profundo|`PyTorch`</br>`TensorFlow`</br>`Keras`</br>`Horovod`</br>`MLFlow`</br>`pandas-ml`</br>`scrapbook`|
|Paquetes de ONNX|`keras2onnx`</br>`onnx`</br>`onnxconverter-common`</br>`skl2onnx`</br>`onnxmltools`|
|Ejemplos del SDK de Python y R para Azure Machine Learning||

Todos los paquetes de Python se instalan en el entorno de **Python 3.6: AzureML**.  

### <a name="installing-packages"></a>Instalación de paquetes

Puede instalar paquetes directamente en Jupyter Notebook o en Rstudio:

* RStudio: use la pestaña **Paquetes**, situada en la parte inferior derecha, o la pestaña **Consola**, situada en la parte superior izquierda.  
* Python: agregue el código de instalación y ejecútelo en una celda de Jupyter Notebook.

También puede acceder a una ventana de terminal de cualquiera de estas formas:

* RStudio: seleccione la pestaña **Terminal**, situada en la parte superior izquierda.
* Jupyter Lab:  seleccione el icono **Terminal** en el encabezado **Otros** de la pestaña Iniciador.
* Jupyter:  seleccione **Nuevo>Terminal** en la parte superior derecha de la pestaña Archivos.
* Conéctese mediante SSH a la máquina virtual.  Después, instale los paquetes de Python en el entorno de **Python 3.6: AzureML**.  Instale los paquetes de R en el entorno de **R**.

## <a name="accessing-files"></a>Acceso a archivos

Los cuadernos y los scripts de R se almacenan en la cuenta de almacenamiento predeterminada del área de trabajo en el recurso compartido de archivos de Azure.  Estos archivos se encuentran en el directorio "Archivos de usuario". Este almacenamiento facilita el uso compartido de cuadernos entre instancias de proceso. La cuenta de almacenamiento también evita que los cuadernos se conserven de forma segura al detener o eliminar una instancia de proceso.

La cuenta de recurso compartido de archivos de Azure del área de trabajo se monta como una unidad en la instancia de proceso. Esta unidad es el directorio de trabajo predeterminado para Jupyter, Jupyter Labs y RStudio. Esto significa que los cuadernos y otros archivos creados en Jupyter, JupyterLab o RStudio se almacenan automáticamente en el recurso compartido de archivos y están disponibles para usarse también en otras instancias de proceso.

Se puede tener acceso a los archivos del recurso compartido de archivos desde todas las instancias de proceso de la misma área de trabajo. Los cambios que se realicen en estos archivos en la instancia de proceso se conservarán de forma confiable en el recurso compartido de archivos.

También puede clonar los ejemplos de Azure Machine Learning más recientes en la carpeta situada en el directorio de archivos de usuario del recurso compartido de archivos del área de trabajo.

La escritura de archivos pequeños puede ser más lenta en las unidades de red que la escritura en el propio disco local de la instancia de proceso.  Si escribe muchos archivos pequeños, pruebe usar un directorio directamente en la instancia de proceso, como un directorio`/tmp`. Tenga en cuenta que estos archivos no serán accesibles desde otras instancias de proceso. 

Puede usar el directorio `/tmp` en la instancia de proceso para los datos temporales.  Pero no escriba grandes archivos de datos en el disco del sistema operativo de la instancia de proceso.  En su lugar, use [almacenes de datos](concept-azure-machine-learning-architecture.md#datasets-and-datastores). Si instaló la extensión git de JupyterLab, también puede provocar una ralentización en el rendimiento de la instancia de proceso.

## <a name="managing-a-compute-instance"></a>Administración de una instancia de proceso

En el área de trabajo de Azure Machine Learning Studio, seleccione **Compute** (Proceso) y, luego, **Compute Instance** (Instancia de proceso) en la parte superior.

![Administración de una instancia de proceso](./media/concept-compute-instance/manage-compute-instance.png)

Puede realizar las siguientes acciones:

* [Crear una instancia de proceso](#create). 
* Actualizar la pestaña instancias de proceso.
* Iniciar, detener y reiniciar una instancia de proceso.  Se paga por la instancia cada vez que se ejecuta. Detenga la instancia de proceso cuando no la use para reducir el costo. Al detener una instancia de proceso, se cancela su asignación. A continuación, inícielo de nuevo cuando lo necesite. 
* Elimine una instancia de proceso.
* Filtre la lista de instancias de proceso a las que creó.  Se trata de las instancias de proceso a las que se puede tener acceso.

Para cada instancia de proceso en el área de trabajo a la que tenga acceso, puede:

* Acceder a Jupyter, JupyterLab y RStudio en la instancia de proceso.
* SSH en la instancia de proceso. El acceso SSH está deshabilitado de forma predeterminada, pero se puede habilitar en el momento de la creación de la instancia de proceso. El acceso SSH se realiza a través del mecanismo de claves pública y privada. La pestaña le proporcionará detalles sobre la conexión SSH, como la dirección IP, el nombre de usuario y el número de puerto.
* Más detalles sobre una instancia de proceso específica, como la dirección IP y la región.

[RBAC](/azure/role-based-access-control/overview) permite controlar qué usuarios del área de trabajo pueden crear, eliminar, iniciar, detener y reiniciar una instancia de proceso. Todos los usuarios del rol colaborador y propietario del área de trabajo pueden crear, eliminar, iniciar, detener y reiniciar las instancias de proceso en el área de trabajo. Sin embargo, solo el creador de una instancia de proceso específica tiene permiso para acceder a Jupyter, JupyterLab y RStudio en esa instancia de proceso. El creador de la instancia de proceso tiene la instancia de proceso dedicada a ellas, tiene acceso raíz y puede realizar el terminal en a través de Jupyter/JupyterLab/RStudio. Solo el usuario creador de la instancia de proceso puede iniciar sesión en ella y todas las acciones usarán la identidad de ese usuario para RBAC y la atribución de ejecuciones de experimentos. El acceso SSH se controla mediante un mecanismo de clave pública-privada.

RBAC puede controlar estas acciones:
* *Microsoft.MachineLearningServices/workspaces/computes/read*
* *Microsoft.MachineLearningServices/workspaces/computes/write*
* *Microsoft.MachineLearningServices/workspaces/computes/delete*
* *Microsoft.MachineLearningServices/workspaces/computes/start/action*
* *Microsoft.MachineLearningServices/workspaces/computes/stop/action*
* *Microsoft.MachineLearningServices/workspaces/computes/restart/action*

### <a name="create-a-compute-instance"></a><a name="create"></a>Crear de una instancia de proceso

En el área de trabajo de Azure Machine Learning Studio, cree una nueva instancia de proceso desde la sección **Proceso** o en la sección **Notebooks** cuando esté listo para ejecutar uno de sus cuadernos.

:::image type="content" source="media/concept-compute-instance/create-compute-instance.png" alt-text="Crear una nueva instancia de proceso":::


|Campo  |Descripción  |
|---------|---------|
|Nombre del proceso     |  <li>El nombre es obligatorio y debe tener una longitud de entre 3 y 24 caracteres.</li><li>Los caracteres válidos son mayúsculas y minúsculas, dígitos y el carácter **-** .</li><li>El nombre debe empezar con una letra</li><li>El nombre debe ser único en todos los procesos existentes dentro de una región de Azure. Verá una alerta si el nombre elegido no es único</li><li>Si se usa el carácter **-** , debe ir seguido de al menos una letra más adelante en el nombre</li>     |
|Tipo de máquina virtual |  Elija CPU o GPU. Este tipo no se puede cambiar después de la creación     |
|Tamaño de la máquina virtual     |  Los tamaños de máquina virtual admitidos podrían estar restringidos en su región. Comprobación de la [lista de disponibilidad](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines)     |
|Habilitación o deshabilitación del acceso SSH     |   El acceso SSH está deshabilitado de forma predeterminada.  El acceso SSH no se puede. cambiar después de la creación. Asegúrese de habilitar el acceso si tiene previsto depurar de forma interactiva con [VS Code Remote](how-to-set-up-vs-code-remote.md)   |
|Configuración avanzada     |  Opcional. Configurar una red virtual. Especifique el **Grupo de recursos**, **Red virtual** y **Subred** para crear la instancia de proceso dentro de una Azure Virtual Network (vnet). Para más información, consulte estos [requisitos de red](how-to-enable-virtual-network.md#compute-instance) para la red virtual.        |

Usted también puede crear una instancia
* Directamente de la [experiencia de cuadernos integrados](tutorial-1st-experiment-sdk-setup.md#azure)
* En Azure Portal
* A partir de una plantilla de Azure Resource Manager. Para ver una plantilla de ejemplo, consulte [Creación una plantilla de instancia de proceso de Azure Machine Learning](https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-compute-create-computeinstance).
* Con SDK de Azure Machine Learning
* Desde la extensión de la [CLI para Azure Machine Learning](reference-azure-machine-learning-cli.md#computeinstance)

Los núcleos dedicados por región por cuota de la familia de máquinas virtuales y la cuota regional total, que se aplica a la creación de la instancia de proceso. está unificada y compartida con la cuota de clúster de proceso de entrenamiento Azure Machine Learning. La detención de la instancia de proceso no libera la cuota para garantizar que pueda reiniciar la instancia de proceso.

## <a name="compute-target"></a>Destino de proceso

Las instancias de proceso se pueden usar como [destino de proceso de entrenamiento](concept-compute-target.md#train) de forma similar a los clústeres de Azure Machine Learning. 

Una instancia de proceso:
* Tiene una cola de trabajo.
* Ejecuta trabajos de forma segura en un entorno de red virtual, sin necesidad de que las empresas abran el puerto SSH. El trabajo se ejecuta en un entorno en contenedor y empaqueta las dependencias del modelo en un contenedor de Docker.
* Puede ejecutar varios trabajos pequeños en paralelo (versión preliminar).  Se pueden ejecutar dos trabajos por núcleo en paralelo mientras el resto de los trabajos se ponen en cola.
* Admite trabajos de entrenamiento distribuidos con varias GPU de un solo nodo.

Puede usar la instancia de proceso como destino de implementación de inferencia local para escenarios de prueba o depuración.

## <a name="what-happened-to-notebook-vm"></a><a name="notebookvm"></a>¿Qué ha ocurrido con la VM de cuaderno?

Las instancias de proceso reemplazan la VM de cuaderno.  

Los archivos de cuaderno almacenados en el recurso compartido de archivos del área de trabajo y los almacenes de datos del área de trabajo serán accesibles desde una instancia de proceso. Sin embargo, los paquetes personalizados que se instalaron previamente en una VM de Notebook deberán volver a instalarse en la instancia de proceso. Las limitaciones de cuota que se aplican a la creación de clústeres de proceso se aplicarán también a la creación de instancias de proceso.

No se pueden crear las nuevas máquinas virtuales de Notebook. Sin embargo, todavía puede tener acceso a las VMs de Notebook que ha creado y utilizarlas con toda funcionalidad. Las instancias de proceso se pueden crear en la misma área de trabajo que las VMs de Notebook existentes.


## <a name="next-steps"></a>Pasos siguientes

 * [Tutorial: Formar su primer modelo de ML](tutorial-1st-experiment-sdk-train.md) muestra cómo usar una instancia de proceso con un cuaderno integrado.
