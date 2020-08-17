---
title: Acción de script para paquetes de Python con Jupyter en Azure HDInsight
description: Instrucciones detalladas sobre cómo usar la acción de script para configurar cuadernos de Jupyter disponibles con clústeres de HDInsight Spark para usar paquetes externos de Python.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020, devx-track-python
ms.date: 04/29/2020
ms.openlocfilehash: 59de3eb2370029ab9edcb609298c7b1fdf5f8ff8
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87873762"
---
# <a name="safely-manage-python-environment-on-azure-hdinsight-using-script-action"></a>Administración segura del entorno de Python en Azure HDInsight mediante la acción de scripts

> [!div class="op_single_selector"]
> * [Uso de magic cell](apache-spark-jupyter-notebook-use-external-packages.md)
> * [Uso de acciones de script](apache-spark-python-package-installation.md)

HDInsight tiene dos instalaciones de Python integradas en el clúster de Spark: Anaconda Python 2.7 y Python 3.5. Es posible que los clientes necesiten personalizar el entorno de Python, como la instalación de paquetes de Python externos u otra versión de Python. En este artículo, se muestra el procedimiento recomendado de administración segura de entornos de Python para clústeres de Apache Spark en HDInsight.

## <a name="prerequisites"></a>Requisitos previos

Un clúster de Apache Spark en HDInsight. Para obtener instrucciones, vea [Creación de clústeres Apache Spark en HDInsight de Azure](apache-spark-jupyter-spark-sql.md). Si aún no tiene un clúster de Spark en HDInsight, puede ejecutar acciones de script durante la creación del clúster. Consulte la documentación sobre [cómo usar acciones de script personalizadas](../hdinsight-hadoop-customize-cluster-linux.md).

## <a name="support-for-open-source-software-used-on-hdinsight-clusters"></a>Soporte técnico para el software de código abierto utilizado en clústeres de HDInsight

El servicio Microsoft Azure HDInsight usa un entorno de tecnologías de código abierto formado en torno a Apache Hadoop. Microsoft Azure proporciona un nivel general de soporte técnico para las tecnologías de código abierto. Para obtener más información, consulte el [sitio web de las preguntas más frecuentes de soporte técnico de Azure](https://azure.microsoft.com/support/faq/). Además, el servicio HDInsight ofrece un nivel adicional de soporte técnico para los componentes incorporados.

Hay dos tipos de componentes de código abierto que están disponibles en el servicio de HDInsight:

|Componente |Descripción |
|---|---|
|Integrada|Estos componentes están instalados previamente en clústeres de HDInsight y proporcionan la funcionalidad básica del clúster. Por ejemplo, el administrador de recursos de Apache Hadoop YARN, el lenguaje de consulta de Apache Hive (HiveQL) y la biblioteca Mahout pertenecen a esta categoría. Hay una lista completa de componentes del clúster disponible en [Novedades en las versiones de clústeres de Apache Hadoop proporcionadas por HDInsight](../hdinsight-component-versioning.md).|
|Personalizado|Como usuario del clúster, puede instalar o usar en la carga de trabajo cualquier componente disponible en la comunidad o que haya creado personalmente.|

> [!IMPORTANT]
> Los componentes proporcionados con el clúster de HDInsight son totalmente compatibles. Soporte técnico de Microsoft ayuda a aislar y a solucionar problemas relacionados con estos componentes.
>
> Los componentes personalizados reciben soporte técnico comercialmente razonable para ayudarle a solucionar el problema. El soporte técnico de Microsoft podría resolver el problema O pedirle que aborde los canales disponibles para las tecnologías de código abierto donde se encuentra la más amplia experiencia para esa tecnología. Por ejemplo, hay diversos sitios de la comunidad que se pueden usar, como el [Página de preguntas y respuestas de Microsoft sobre HDInsight](https://docs.microsoft.com/answers/topics/azure-hdinsight.html),`https://stackoverflow.com`. Los proyectos de Apache también tienen sitios de proyecto en `https://apache.org`.

## <a name="understand-default-python-installation"></a>Descripción de la instalación predeterminada de Python

El clúster de HDInsight Spark se crea con la instalación de Anaconda. Hay dos instalaciones de Python en el clúster: Anaconda Python 2.7 y Python 3.5 En la tabla siguiente se muestra la configuración predeterminada de Python para Spark, Livy y Jupyter.

|Configuración |Python 2.7|Python 3.5|
|----|----|----|
|Path|/usr/bin/anaconda/bin|/usr/bin/anaconda/envs/py35/bin|
|Versión de Spark|Valor predeterminado establecido en 2.7|N/D|
|Versión de Livy|Valor predeterminado establecido en 2.7|N/D|
|Jupyter|Kernel de PySpark|Kernel de PySpark3|

## <a name="safely-install-external-python-packages"></a>Instalación segura de paquetes externos de Python

El clúster de HDInsight depende del entorno integrado de Python, tanto Python 2.7 como Python 3.5. La instalación directa de paquetes personalizados en esos entornos integrados predeterminados puede producir cambios inesperados en la versión de la biblioteca y generar más interrupciones en el clúster. Para instalar de forma segura paquetes externos personalizados de Python para las aplicaciones de Spark, siga los pasos que se indican a continuación.

1. Cree un entorno virtual de Python mediante Conda. Un entorno virtual proporciona un espacio aislado para sus proyectos sin interrumpir otros. Al crear el entorno virtual de Python, puede especificar la versión de Python que quiere usar. Sin embargo, seguirá siendo necesario crear el entorno virtual, aunque quiera usar Python 2.7 y 3.5. Este requisito tiene como fin asegurarse de que no se interrumpa el entorno predeterminado del clúster. Ejecute las acciones de script en el clúster para todos los nodos con el siguiente script para crear un entorno virtual de Python.

    -   `--prefix` especifica una ruta de acceso donde reside un entorno virtual de Conda. Hay varias configuraciones que deben seguirse cambiando en función de la ruta de acceso especificada aquí. En este ejemplo, usamos py35new, ya que el clúster tiene un entorno virtual existente denominado py35.
    -   `python=` especifica la versión de Python para el entorno virtual. En este ejemplo, usamos la versión 3.5, la misma versión que el clúster creado anteriormente. También puede usar otras versiones de Python para crear el entorno virtual.
    -   `anaconda` especifica el package_spec como anaconda para instalar paquetes de Anaconda en el entorno virtual.
    
    ```bash
    sudo /usr/bin/anaconda/bin/conda create --prefix /usr/bin/anaconda/envs/py35new python=3.5 anaconda --yes
    ```

2. Instale los paquetes externos de Python en el entorno virtual creado si es necesario. Ejecute las acciones de script en el clúster para todos los nodos con el siguiente script para instalar los paquetes externos de Python. Debe tener el privilegio sudo aquí para escribir archivos en la carpeta del entorno virtual.

    Buscar en el [índice de paquetes](https://pypi.python.org/pypi) la lista completa de paquetes que están disponibles. También puede obtener una lista de paquetes disponibles de otras fuentes. Por ejemplo, puede instalar paquetes que están disponibles a través de [conda-forge](https://conda-forge.org/feedstocks/).

    Use el comando siguiente si quiere instalar una biblioteca con su versión más reciente:

    - Usar el canal de Conda:

        -   `seaborn` es el nombre del paquete que desea instalar.
        -   `-n py35new` especifica el nombre del entorno virtual que acaba de crearse. Asegúrese de cambiar el nombre correspondiente en función de la creación del entorno virtual.

        ```bash
        sudo /usr/bin/anaconda/bin/conda install seaborn -n py35new --yes
        ```

    - O bien use el repositorio de PyPi y cambie `seaborn` y `py35new` según corresponda:
        ```bash
        sudo /usr/bin/anaconda/env/py35new/bin/pip install seaborn
        ```

    Use el comando siguiente si quiere instalar una biblioteca con una versión específica:

    - Usar el canal de Conda:

        -   `numpy=1.16.1` es el nombre y la versión del paquete que quiere instalar.
        -   `-n py35new` especifica el nombre del entorno virtual que acaba de crearse. Asegúrese de cambiar el nombre correspondiente en función de la creación del entorno virtual.

        ```bash
        sudo /usr/bin/anaconda/bin/conda install numpy=1.16.1 -n py35new --yes
        ```

    - O bien use el repositorio de PyPi y cambie `numpy==1.16.1` y `py35new` según corresponda:

        ```bash
        sudo /usr/bin/anaconda/env/py35new/bin/pip install numpy==1.16.1
        ```

    Si no conoce el nombre del entorno virtual, puede utilizar SSH en el nodo de encabezado del clúster y ejecutar `/usr/bin/anaconda/bin/conda info -e` para mostrar todos los entornos virtuales.

3. Cambie las configuraciones de Spark y Livy, y apunte al entorno virtual creado.

    1. Abra la UI de Ambari, vaya a la página Spark2, pestaña Configs.

        ![Cambiar la configuración de Spark y Livy a través de Ambari](./media/apache-spark-python-package-installation/ambari-spark-and-livy-config.png)

    2. Expanda Advanced livy2-env y agregue las siguientes instrucciones al final. Si instaló el entorno virtual con otro prefijo, cambie la ruta de acceso correspondiente.

        ```bash
        export PYSPARK_PYTHON=/usr/bin/anaconda/envs/py35new/bin/python
        export PYSPARK_DRIVER_PYTHON=/usr/bin/anaconda/envs/py35new/bin/python
        ```

        ![Cambiar la configuración de Livy a través de Ambari](./media/apache-spark-python-package-installation/ambari-livy-config.png)

    3. Expanda Advanced spark2-env y reemplace la instrucción de exportación PYSPARK_PYTHON existente al final. Si instaló el entorno virtual con otro prefijo, cambie la ruta de acceso correspondiente.

        ```bash
        export PYSPARK_PYTHON=${PYSPARK_PYTHON:-/usr/bin/anaconda/envs/py35new/bin/python}
        ```

        ![Cambiar la configuración de Spark a través de Ambari](./media/apache-spark-python-package-installation/ambari-spark-config.png)

    4. Guarde los cambios y reinicie los servicios afectados. Estos cambios necesitan el reinicio del servicio Spark2. La UI de Ambari le mostrará un aviso de reinicio obligatorio, haga clic en Restart para reiniciar todos los servicios afectados.

        ![Cambiar la configuración de Spark a través de Ambari](./media/apache-spark-python-package-installation/ambari-restart-services.png)

4. Si desea usar el nuevo entorno virtual creado en Jupyter, cambie las configuraciones de Jupyter y reinicie Jupyter. Ejecute las acciones de script en todos los nodos de encabezado con la siguiente instrucción para apuntar Jupyter al nuevo entorno virtual creado. Asegúrese de modificar la ruta de acceso al prefijo que especificó para el entorno virtual. Después de ejecutar esta acción de script, reinicie el servicio de Jupyter a través de la UI de Ambari para que este cambio esté disponible.

    ```bash
    sudo sed -i '/python3_executable_path/c\ \"python3_executable_path\" : \"/usr/bin/anaconda/envs/py35new/bin/python3\"' /home/spark/.sparkmagic/config.json
    ```

    Puede volver a confirmar el entorno de Python en Jupyter Notebook ejecutando el código siguiente:

    ![Comprobar la versión de Python en Jupyter Notebook](./media/apache-spark-python-package-installation/check-python-version-in-jupyter.png)

## <a name="known-issue"></a>Problema conocido

Hay un error conocido para la versión de Anaconda `4.7.11`, `4.7.12` y `4.8.0`. Si observa que las acciones de script dejan de responder en `"Collecting package metadata (repodata.json): ...working..."` y muestran el error `"Python script has been killed due to timeout after waiting 3600 secs"`. Puede descargar [este script](https://gregorysfixes.blob.core.windows.net/public/fix-conda.sh) y ejecutarlo como acciones de script en todos los nodos para corregir el problema.

Para comprobar la versión de Anaconda, puede aplicar SSH en el nodo de encabezado del clúster y ejecutar `/usr/bin/anaconda/bin/conda --v`.

## <a name="next-steps"></a>Pasos siguientes

* [Información general: Apache Spark en Azure HDInsight](apache-spark-overview.md)
* [Paquetes externos con cuadernos de Jupyter en Apache Spark](apache-spark-jupyter-notebook-use-external-packages.md)
* [Track and debug jobs running on an Apache Spark cluster in HDInsight (Seguimiento y depuración de trabajos que se ejecutan en un clúster de Apache Spark en HDInsight)](apache-spark-job-debugging.md)
