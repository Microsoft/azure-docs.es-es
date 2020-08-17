---
title: Configurar un entorno de desarrollo de Python
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo configurar un entorno de desarrollo para Azure Machine Learning. Use entornos Conda para crear archivos de configuración y configurar su propio servidor de cuadernos basado en la nube, Jupyter Notebook, Azure Databricks, IDE, editores de código y Data Science Virtual Machine.
services: machine-learning
author: rastala
ms.author: roastala
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.date: 12/27/2019
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 09d82c4834e50e0dff74edb5ccd14287736b937e
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87844480"
---
# <a name="configure-a-development-environment-for-azure-machine-learning"></a>Configurar un entorno de desarrollo para Azure Machine Learning
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo obtendrá información acerca de cómo configurar un entorno de desarrollo para trabajar con Azure Machine Learning. Azure Machine Learning es independiente de la plataforma. El único requisito imprescindible para el entorno de desarrollo es Python 3. También se recomienda un entorno aislado, como Anaconda o Virtualenv.

En la tabla siguiente se muestra cada entorno de desarrollo que se trata en este artículo, junto con las ventajas y desventajas.

| Entorno | Ventajas | Desventajas |
| --- | --- | --- |
| [Instancia de proceso de Azure Machine Learning basado en la nube (versión preliminar)](#compute-instance) | Manera más fácil de empezar. El SDK completo ya está instalado en la máquina virtual del área de trabajo y los tutoriales de cuaderno están clonados previamente y listos para ejecutarse. | Falta de control sobre el entorno de desarrollo y las dependencias. Costo adicional por la máquina virtual Linux (la máquina virtual se puede detener cuando no se use para evitar cargos). Consulte los [detalles de los precios](https://azure.microsoft.com/pricing/details/virtual-machines/linux/). |
| [Entorno local](#local) | Control total del entorno de desarrollo y las dependencias. Funciona con cualquier herramienta de compilación, entorno o IDE de su elección. | Tarda más tiempo en comenzar. Se deben instalar los paquetes de SDK necesarios y también debe instalarse un entorno si aún no tiene uno. |
| [Azure Databricks](#aml-databricks) | Perfecto para ejecutar flujos de trabajo de aprendizaje automático intensivos y a gran escala en la plataforma escalable de Apache Spark. | Excesivo para aprendizaje automático experimental o experimentos y flujos de trabajo a pequeña escala. Costo adicional por Azure Databricks. Consulte los [detalles de los precios](https://azure.microsoft.com/pricing/details/databricks/). |
| [Data Science Virtual Machine (DSVM)](#dsvm) | De forma parecida a la instancia de proceso basada en la nube (con Python y el SDK preinstalados), pero con herramientas adicionales de ciencia de datos y aprendizaje automático conocidas instaladas previamente. Fácil de escalar y combinar con otras herramientas y flujos de trabajo personalizados. | Una experiencia de inicio más lenta en comparación con la instancia de proceso basada en la nube. |

En este artículo también se proporcionan sugerencias de uso adicionales para las siguientes herramientas:

* [Jupyter Notebooks](#jupyter): si ya usa instancias de Jupyter Notebook, el SDK incluye algunos elementos adicionales que debe instalar.

* [Visual Studio Code](#vscode): Si usa Visual Studio Code, la [extensión de Azure Machine Learning](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.vscode-ai) incluye un amplio soporte de lenguaje para Python, así como características para que trabajar con Azure Machine Learning sea mucho más cómodo y productivo.

## <a name="prerequisites"></a>Requisitos previos

Un área de trabajo de Azure Machine Learning. Para crear el área de trabajo, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md). Un área de trabajo es todo lo que necesita para empezar a trabajar con su propio [servidor de cuadernos basado en la nube](#compute-instance), una instancia de [DSVM](#dsvm) o [Azure Databricks](#aml-databricks).

Para instalar el entorno de SDK para el [equipo local](#local), el [servidor de Jupyter Notebook](#jupyter) o [Visual Studio Code](#vscode), también necesita:

- El administrador de paquetes de [Anaconda](https://www.anaconda.com/download/) o [Miniconda](https://conda.io/miniconda.html).

- En Linux o en Mac OS, es necesario tener el shell de bash.

    > [!TIP]
    > Si usa Linux o Mac OS y un shell distinto de bash (por ejemplo, zsh), es posible que reciba errores al ejecutar algunos comandos. Para solucionar este problema, use el comando `bash` para iniciar un nuevo shell de bash y ejecutar sus comandos.

- En Windows, se necesita el símbolo del sistema o la solicitud de Anaconda (que se instala con Anaconda y Miniconda).

## <a name="your-own-cloud-based-compute-instance"></a><a id="compute-instance"></a>Su propia instancia de proceso basada en la nube

La [instancia de proceso (versión preliminar)](concept-compute-instance.md) de Azure Machine Learning es una estación de trabajo de Azure segura y basada en la nube que proporciona a los científicos de datos un servidor de Jupyter Notebook, JupyterLab y un entorno de ML completamente preparado.

No hay nada que instalar o configurar para una instancia de proceso.  Cree una en cualquier momento desde el área de trabajo de Azure Machine Learning. Basta con que proporcione un nombre y especifique un tipo de máquina virtual de Azure. Pruébelo ahora con este [Tutorial: Configuración del entorno y el área de trabajo](tutorial-1st-experiment-sdk-setup.md).

Para más información sobre las instancias de proceso, incluido cómo instalar paquetes, vea [Instancias de proceso](concept-compute-instance.md).

Para detener los cargos de proceso generados, [detenga la instancia de proceso](tutorial-1st-experiment-sdk-train.md#clean-up-resources).

## <a name="data-science-virtual-machine"></a><a id="dsvm"></a>Data Science Virtual Machine

DSVM es una imagen de máquina virtual personalizada. Está diseñada para realizar trabajos de ciencia de datos que estén preconfigurados con:

  - Paquetes, como TensorFlow, PyTorch, Scikit-learn, XGBoost y el SDK de Azure Machine Learning
  - Herramientas de ciencia de datos populares, como Spark Standalone o Drill
  - Herramientas de Azure, como la CLI de Azure, AzCopy y el Explorador de Storage
  - Entornos de desarrollo integrado (IDE), como Visual Studio Code y PyCharm
  - Servidor de Jupyter Notebook

El SDK de Azure Machine Learning funciona en la versión de Ubuntu o de Windows de DSVM. Pero si también va a usar DSVM como destino de proceso, solo se admite Ubuntu.

Para usar DSVM como entorno de desarrollo:

1. Cree una instancia de DSVM en cualquiera de los siguientes entornos:

    * Azure Portal:

        * [Crear una instancia de Data Science Virtual Machine de Ubuntu](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro)

        * [Crear una instancia de Data Science Virtual Machine de Windows](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/provision-vm)

    * La CLI de Azure:

        > [!IMPORTANT]
        > * Si usa la CLI de Azure, primero debe iniciar sesión en su suscripción de Azure con el comando `az login`.
        >
        > * Al usar los comandos en este paso, debe proporcionar un nombre de grupo de recursos, un nombre para la máquina virtual, un nombre de usuario y una contraseña.

        * Para crear una instancia de Data Science Virtual Machine de Ubuntu, use el siguiente comando:

            ```azurecli-interactive
            # create a Ubuntu DSVM in your resource group
            # note you need to be at least a contributor to the resource group in order to execute this command successfully
            # If you need to create a new resource group use: "az group create --name YOUR-RESOURCE-GROUP-NAME --location YOUR-REGION (For example: westus2)"
            az vm create --resource-group YOUR-RESOURCE-GROUP-NAME --name YOUR-VM-NAME --image microsoft-dsvm:linux-data-science-vm-ubuntu:linuxdsvmubuntu:latest --admin-username YOUR-USERNAME --admin-password YOUR-PASSWORD --generate-ssh-keys --authentication-type password
            ```

        * Para crear una instancia de Data Science Virtual Machine de Windows, use el siguiente comando:

            ```azurecli-interactive
            # create a Windows Server 2016 DSVM in your resource group
            # note you need to be at least a contributor to the resource group in order to execute this command successfully
            az vm create --resource-group YOUR-RESOURCE-GROUP-NAME --name YOUR-VM-NAME --image microsoft-dsvm:dsvm-windows:server-2016:latest --admin-username YOUR-USERNAME --admin-password YOUR-PASSWORD --authentication-type password
            ```

2. El SDK de Azure Machine Learning ya está instalado en DSVM. Para usar el entorno de Conda que contiene el SDK, use uno de los siguientes comandos:

    * Para la instancia de DSVM de Ubuntu:

        ```bash
        conda activate py36
        ```

    * Para la instancia de DSVM de Windows:

        ```bash
        conda activate AzureML
        ```

1. Para verificar que puede acceder al SDK y comprobar la versión, use el código de Python siguiente:

    ```python
    import azureml.core
    print(azureml.core.VERSION)
    ```

1. Para configurar la instancia de DSVM que usará en el área de trabajo de Azure Machine Learning, consulte la sección [Crear un archivo de configuración del área de trabajo](#workspace).

Consulte [Data Science Virtual Machine](https://azure.microsoft.com/services/virtual-machines/data-science-virtual-machines/) para obtener más información.

## <a name="local-computer"></a><a id="local"></a>Equipo local

Cuando use un equipo local (que también podría ser una máquina virtual remota), cree un entorno de Anaconda e instale el SDK. Este es un ejemplo:

1. Descargue e instale [Anaconda](https://www.anaconda.com/distribution/#download-section) (versión 3.7 de Python) si aún no lo tiene.

1. Abra un símbolo del sistema de Anaconda y cree un entorno con los comandos siguientes:

    Ejecute el comando siguiente para crear el entorno.

    ```bash
    conda create -n myenv python=3.7.7
    ```

    Después, active el entorno.

    ```bash
    conda activate myenv
    ```

    En este ejemplo se crea un entorno con Python 3.7.7, pero se puede elegir cualquier versión secundaria. La compatibilidad con el SDK no puede garantizarse con determinadas versiones principales (se recomienda la 3.5 y versiones posteriores). Además, se aconseja probar una versión principal o secundaria diferente en el entorno de Anaconda si se experimentan errores. La creación del entorno tardará unos minutos mientras se descargan los componentes y los paquetes.

1. Ejecute los comandos siguientes en el nuevo entorno para habilitar los kernels de IPython específicos del entorno. Así se garantizará el comportamiento esperado de importación de paquetes y kernel al trabajar con cuadernos de Jupyter Notebook en entornos de Anaconda:

    ```bash
    conda install notebook ipykernel
    ```

    Después, ejecute el siguiente comando para crear el kernel:

    ```bash
    ipython kernel install --user --name myenv --display-name "Python (myenv)"
    ```

1. Use los comandos siguientes para instalar los paquetes:

    Este comando instala el SDK básico de Azure Machine Learning con elementos adicionales de cuadernos y `automl`. El elemento adicional `automl` es una instalación de gran tamaño y puede quitarse de los corchetes si no se van a ejecutar experimentos automatizados de aprendizaje automático. El elemento adicional `automl` también incluye de forma predeterminada el SDK de preparación de datos de Azure Machine Learning como una dependencia.

    ```bash
    pip install azureml-sdk[notebooks,automl]
    ```

   > [!NOTE]
   > * Si recibe un mensaje que indica que PyYAML no se puede desinstalar, use el comando siguiente en su lugar:
   >
   >   `pip install --upgrade azureml-sdk[notebooks,automl] --ignore-installed PyYAML`
   >
   > * A partir de macOS Catalina, zsh (Z shell) es el shell de inicio de sesión predeterminado y el shell interactivo. En zsh, use el siguiente comando que escapa los corchetes con el carácter "\\" (barra diagonal inversa):
   >
   >   `pip install --upgrade azureml-sdk\[notebooks,automl\]`

   El SDK tardará varios minutos en instalarse. Consulte la [guía de instalación](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py) para más información sobre las opciones de instalación.

1. Instale otros paquetes para experimentar con el aprendizaje automático.

    Use cualquiera de los comandos siguientes y reemplace *\<new package>* por el paquete que quiera instalar. La instalación de paquetes a través de `conda install` requiere que el paquete forme parte de los canales actuales (se pueden agregar nuevos canales en la nube de Anaconda).

    ```bash
    conda install <new package>
    ```

    También puede instalar paquetes a través de `pip`.

    ```bash
    pip install <new package>
    ```

### <a name="jupyter-notebooks"></a><a id="jupyter"></a>Jupyter Notebook

Las instancias de Jupyter Notebook forman parte de [Jupyter Project](https://jupyter.org/). Proporcionan una experiencia de codificación interactiva en la que se crean documentos que mezclan código activo con texto narrativo y gráficos. Jupyter Notebook proporciona una excelente manera de compartir los resultados con otros usuarios, ya que le permite guardar la salida de las secciones de código en el documento. Puede instalar cuadernos de Jupyter Notebook en una variedad de plataformas.

Mediante el procedimiento descrito en la sección [Equipo local](#local) se instalan los componentes necesarios para ejecutar cuadernos de Jupyter Notebook en un entorno de Anaconda.

Para habilitar estos componentes en el entorno de Jupyter Notebook:

1. Abra un símbolo del sistema de Anaconda y active el entorno.

    ```bash
    conda activate myenv
    ```

1. Clone [el repositorio de GitHub](https://aka.ms/aml-notebooks) para un conjunto de cuadernos de ejemplo.

    ```bash
    git clone https://github.com/Azure/MachineLearningNotebooks.git
    ```

1. Inicie el servidor de Jupyter Notebook con el comando siguiente:

    ```bash
    jupyter notebook
    ```

1. Para comprobar si Jupyter Notebook puede usar el SDK, cree un cuaderno **Nuevo**, seleccione **Python 3** como kernel y ejecute el comando siguiente en una celda del cuaderno:

    ```python
    import azureml.core
    azureml.core.VERSION
    ```

1. Si encuentra problemas al importar los módulos y recibe un error de tipo `ModuleNotFoundError`, asegúrese de que el kernel de Jupyter está conectado a la ruta de acceso correcta del entorno. Para ello, ejecute el código siguiente en una celda del cuaderno.

    ```python
    import sys
    sys.path
    ```

1. Para configurar la instancia de Jupyter Notebook que usará en el área de trabajo de Azure Machine Learning, consulte la sección [Crear un archivo de configuración del área de trabajo](#workspace).

### <a name="visual-studio-code"></a><a id="vscode"></a>Visual Studio Code

Visual Studio Code es un editor de código multiplataforma muy popular que admite un amplio conjunto de lenguajes de programación y herramientas a través de extensiones disponibles en [Visual Studio marketplace](https://marketplace.visualstudio.com/vscode). La [extensión de Azure Machine Learning](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.vscode-ai) instala la [extensión de Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python) para poder codificar en todo tipo de entornos Python (virtual, Anaconda, etc.). Asimismo, proporciona características útiles para trabajar con recursos de Azure Machine Learning y ejecutar experimentos de Azure Machine Learning sin tener que salir de Visual Studio Code.

Para usar Visual Studio Code para el desarrollo:

1. Para instalar la extensión de Azure Machine Learning para Visual Studio Code, consulte [Azure Machine Learning](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.vscode-ai).

    Para obtener más información, consulte [Usar Azure Machine Learning para Visual Studio Code](tutorial-setup-vscode-extension.md).

1. Para obtener información acerca de cómo usar Visual Studio Code para el desarrollo de Python, consulte [Introducción a Python en VSCode](https://code.visualstudio.com/docs/python/python-tutorial).

    - Para seleccionar el entorno de SDK Python que contiene el SDK, abra VS Code y seleccione Ctrl + Shift + P (Linux y Windows) o Comando + Shift + P (Mac).
        - Se abrirá la __Paleta de comandos__.

    - Escriba __Python: seleccionar intérprete__, y luego seleccione el entorno apropiado

1. Para confirmar que puede usar el SDK, cree un nuevo archivo de Python (.py) que contenga el código siguiente:

    ```python
    #%%
    import azureml.core
    azureml.core.VERSION
    ```
    Ejecute este código haciendo clic en la opción "Ejecutar celda" de CodeLens o simplemente presione Mayús + Entrar.
<a name="aml-databricks"></a>

## <a name="azure-databricks"></a>Azure Databricks
Azure Databricks es un entorno basado en Apache Spark de la nube de Azure. Proporciona un entorno de colaboración basado en cuadernos con un clúster de proceso basado en CPU o GPU.

Funcionamiento de Azure Databricks con Azure Machine Learning:
+ Puede entrenar un modelo con Spark MLlib e implementar el modelo en ACI o AKS desde Azure Databricks.
+ También puede usar funcionalidades de [aprendizaje automático automatizado](concept-automated-ml.md) en un SDK de Azure ML especial con Azure Databricks.
+ Puede usar Azure Databricks como destino de proceso desde una [canalización de Azure Machine Learning](concept-ml-pipelines.md).

### <a name="set-up-your-databricks-cluster"></a>Configuración del clúster de Databricks

Cree un [clúster de Databricks](https://docs.microsoft.com/azure/azure-databricks/quickstart-create-databricks-workspace-portal). Algunas opciones solo se aplican si instala el SDK para el aprendizaje automático automatizado en Databricks.
**La operación de creación del clúster tardará unos minutos.**

Use estos valores de configuración:

| Configuración |Se aplica a| Value |
|----|---|---|
| Nombre del clúster |Siempre| nombredelclúster |
| Entorno de tiempo de ejecución de Databricks |Siempre|Tiempo de ejecución no de ML 6.5 (Scala 2.11, Spark 2.4.3) |
| Versión de Python |Siempre| 3 |
| Trabajos |Siempre| 2 o más |
| Tipos de máquinas virtuales con nodos de trabajo <br>(determina el número máximo de iteraciones simultáneas) |Machine Learning Automatizado<br>solo| Se prefieren las máquinas virtuales optimizadas para memoria |
| Habilitar escalado automático |Machine Learning Automatizado<br>solo| Desactivar |

Espere hasta que el clúster se ejecute antes de continuar.

### <a name="install-the-correct-sdk-into-a-databricks-library"></a>Instalación del SDK correcto en una biblioteca de Databricks
Una vez que se esté ejecutando el clúster, [cree una biblioteca](https://docs.databricks.com/user-guide/libraries.html#create-a-library) para adjuntar al clúster el paquete del SDK adecuado de Azure Machine Learning.

1. Haga clic con el botón derecho en la carpeta del área de trabajo actual en la que quiere almacenar la biblioteca. Seleccione **Crear** > **Biblioteca**.

1. Elija **solo una** opción (no se admite ninguna otra instalación de SDK).

   |Elementos adicionales del&nbsp;paquete&nbsp;del SDK|Source|Nombre de&nbsp;PyPi&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|
   |----|---|---|
   |Para Databricks| Cargar un huevo o PyPi de Python | azureml-sdk[databricks]|
   |Para Databricks con<br> funcionalidades de ML automatizado| Cargar un huevo o PyPi de Python | azureml-sdk[automl]|

   > [!Warning]
   > No se pueden instalar otros elementos adicionales del SDK. Elija solo una de las opciones anteriores: [databricks] o [automl].

   * No seleccione **Attach automatically to all clusters** (Asociar automáticamente a todos los clústeres).
   * Seleccione **Asociar** junto al nombre del clúster.

1. Supervise los errores hasta que el estado cambie a **Asociado**, lo que podría tardar varios minutos.  Si se produce un error en este paso:

   Pruebe a reiniciar el clúster de la manera siguiente:
   1. En el panel izquierdo, seleccione **Clústeres**.
   1. En la tabla, seleccione el nombre del clúster.
   1. En la ficha **Bibliotecas**, seleccione **Reiniciar**.

   También debe tener en cuenta lo siguiente:
   + En la configuración de AutoML, al usar Azure Databricks, agregue los siguientes parámetros:
       1. ```max_concurrent_iterations``` se basa en el número de nodos de trabajo del clúster.
        2. ```spark_context=sc``` se basa en el contexto de Spark predeterminado.
   + O bien, si tiene una versión anterior del SDK, anule la selección de este desde las bibliotecas instaladas del clúster y muévalo a la Papelera. Instale la nueva versión del SDK y reinicie el clúster. Si hay un problema después del reinicio, desasocie y asocie nuevamente el clúster.

Si la instalación fue correcta, la biblioteca importada debería tener un aspecto similar a uno de los ejemplos siguientes:

SDK para Databricks **_sin_** aprendizaje automático automatizado ![SDK de Azure Machine Learning para Databricks](./media/how-to-configure-environment/amlsdk-withoutautoml.jpg)

SDK para Databricks **CON** aprendizaje automático automatizado ![SDK con aprendizaje automático automatizado instalado en Databricks](./media/how-to-configure-environment/automlonadb.png)

### <a name="start-exploring"></a>Comienzo de la exploración

Pruebe lo siguiente:
+ Aunque hay muchos cuadernos de ejemplo disponibles, **solo [estos cuadernos de ejemplo](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/azure-databricks) funcionan con Azure Databricks.**

+ Importe estos ejemplos directamente desde el área de trabajo. Observe a continuación: ![Selección de Importar](./media/how-to-configure-environment/azure-db-screenshot.png)
![Panel Importar](./media/how-to-configure-environment/azure-db-import.png)

+ Obtenga información sobre cómo [crear una canalización con Databricks como proceso de entrenamiento](how-to-create-your-first-pipeline.md).

## <a name="create-a-workspace-configuration-file"></a><a id="workspace"></a>Crear un archivo de configuración del área de trabajo

El archivo de configuración del área de trabajo es un archivo JSON que le indica al SDK cómo comunicarse con el área de trabajo de Azure Machine Learning. El archivo se denomina *config.json* y tiene el formato siguiente:

```json
{
    "subscription_id": "<subscription-id>",
    "resource_group": "<resource-group>",
    "workspace_name": "<workspace-name>"
}
```

Este archivo JSON debe estar en la estructura de directorios que contiene los scripts de Python o las instancias de Jupyter Notebook. Puede estar en el mismo directorio, en un subdirectorio denominado *.azureml* o en un directorio principal.

Para utilizar este archivo desde el código, utilice `ws=Workspace.from_config()`. Este código carga la información del archivo y se conecta a su área de trabajo.

Hay tres formas de crear el archivo de configuración:

* **Use [ws.write_config](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py)** : para escribir un archivo *config.json*. Este archivo contiene información de configuración para el área de trabajo. Puede descargar o copiar el archivo *config.json* en otros entornos de desarrollo.

* **Descargue el archivo**: En [Azure Portal](https://ms.portal.azure.com), seleccione **Descargar config.json** desde la sección **Información general** del área de trabajo.

     ![Azure portal](./media/how-to-configure-environment/configure.png)

* **Cree el archivo mediante programación**: En el fragmento de código siguiente se muestra cómo conectarse a un área de trabajo proporcionando el id. de suscripción, el grupo de recursos y el nombre del área de trabajo. A continuación, se guarda la configuración del área de trabajo en el archivo:

    ```python
    from azureml.core import Workspace

    subscription_id = '<subscription-id>'
    resource_group  = '<resource-group>'
    workspace_name  = '<workspace-name>'

    try:
        ws = Workspace(subscription_id = subscription_id, resource_group = resource_group, workspace_name = workspace_name)
        ws.write_config()
        print('Library configuration succeeded')
    except:
        print('Workspace not found')
    ```

    Este código escribe el archivo de configuración en el archivo *.azureml/config.json*.

## <a name="next-steps"></a>Pasos siguientes

- [Entrenar un modelo](tutorial-train-models-with-aml.md) en Azure Machine Learning con el conjunto de datos de MNIST
- Consultar las referencias del [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py)
